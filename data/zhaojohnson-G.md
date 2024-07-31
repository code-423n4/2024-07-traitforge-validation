[G1] In mintWithBudget(), we can mint multiple NFTs in one transaction with the specified budget. When we mint one NFT, we will transfer mint price to the NukeFund via `_distributeFunds()`. If one whale want to mint a lot NFTs, there will be a lot of inner interaction between TraitForgeNft and NukeFund. We can consider to distribute funds once after we confirm the total mint prices.

[G2] When NukeFunds receives some funds, NukeFund will split 10% to the DevFund. 
The problem is that if `msg.value % totalDevWeight != 0`, we will return back some funds. 
Considering one scenario:
- The `totalDevWeight` is 100000. 
- NukeFund receive 999999 ether, and send 99999 to DevFund.
- DevFund will send back the whole 99999 back to the NukeFund.
- NukeFund will split funds again and transfer 9999 to DevFund.
- DevFund will return the 9999 back to the NukeFund.
- NukeFund will split funds again and transfer 999 to DevFund.
- DevFund will return the 999 back to the NukeFund.
- NukeFund will split funds again and transfer 99 to DevFund.
- DevFund will return 99 back to the NukeFund.
- NukeFund will split funds again and transfer 9 to DevFund.
- DevFund will return 9 back to the NukeFund.
- NukeFund will split funds again and transfer 0 to DevFund.
- DevFund will finish because the remain will be 0.
All these next loop are non-sense, we can consider to check `devShare` should be not less than DevFund's `totalDevWeight` in NukeFund.
```javascript
  receive() external payable {
    uint256 devShare = msg.value / taxCut; // Calculate developer's share (10%)
    uint256 remainingFund = msg.value - devShare; // Calculate remaining funds to add to the fund

    fund += remainingFund; // Update the fund balance
    // If airdrop not start, transfer to developer directly.
    if (!airdropContract.airdropStarted()) {
      (bool success, ) = devAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    }
    ...
  }
  receive() external payable {
    if (totalDevWeight > 0) {
      // @audit gas optimization.
      uint256 amountPerWeight = msg.value / totalDevWeight;
      // Return some dust ether back to the funder.
      uint256 remaining = msg.value - (amountPerWeight * totalDevWeight);
      totalRewardDebt += amountPerWeight;
      if (remaining > 0) {
        (bool success, ) = payable(owner()).call{ value: remaining }('');
        require(success, 'Failed to send Ether to owner');
      } 
    ...
   }
``` 