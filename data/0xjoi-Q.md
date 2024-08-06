### Dangerous strict equalities

L1 - ❌ EntropyGenerator.deriveTokenParameters(uint256,uint256) (contracts/EntropyGenerator/EntropyGenerator.sol:136-161) 
uses a dangerous strict equality:    

```isForger = role == 0``` 

(contracts/EntropyGenerator/EntropyGenerator.sol#158)Recommendation: Don't use strict equality to determine if an account has enough Ether or tokens.

code link -> https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L158C5-L158C13


### Dangerous strict equalities

L2 - ❌ EntityForging._resetForgingCountIfNeeded(uint256) (contracts/EntityForging/EntityForging.sol:201-209) uses a dangerous strict equality:   

```lastForgeResetTimestamp[tokenId] == 0```

(contracts/EntityForging/EntityForging.sol#203)Recommendation: Don't use strict equality to determine if an account has enough Ether or tokens.

code link -> https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L201


### Devfund.recieve() 

L3 - Reentrancy Affecting Events Ordering

```solidity
  receive() external payable {
    if (totalDevWeight > 0) {
      // @audit fishy
      uint256 amountPerWeight = msg.value / totalDevWeight;
      uint256 remaining = msg.value - (amountPerWeight * totalDevWeight);
      totalRewardDebt += amountPerWeight;
      if (remaining > 0) {
        (bool success, ) = payable(owner()).call{ value: remaining }('');
        require(success, 'Failed to send Ether to owner');
      }
    } else {
      (bool success, ) = payable(owner()).call{ value: msg.value }('');
      require(success, 'Failed to send Ether to owner');
    }
    emit FundReceived(msg.sender, msg.value);
  }

```



### NukeFund.recieve()


L4 - Reentrancy Affecting Events Ordering

```solidity
  receive() external payable {

    // @audit check rentrancy?
    uint256 devShare = msg.value / taxCut; // Calculate developer's share (10%)
    uint256 remainingFund = msg.value - devShare; // Calculate remaining funds to add to the fund

    fund += remainingFund; // Update the fund balance

    if (!airdropContract.airdropStarted()) {
      (bool success, ) = devAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    } else if (!airdropContract.daoFundAllowed()) {
      (bool success, ) = payable(owner()).call{ value: devShare }('');
      // @audit rentrancy??
      require(success, 'ETH send failed');
    } else {
      (bool success, ) = daoAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    }

    emit FundReceived(msg.sender, msg.value); // Log the received funds
    emit FundBalanceUpdated(fund); // Update the fund balance
  }
```


### EntropyGenerator.initializeAlphaIndices() 

L5 - EntropyGenerator.initializeAlphaIndices() (contracts/EntropyGenerator/EntropyGenerator.sol#206-216) uses a weak PRNG: "numberIndexSelection = hashValue % 13 (contracts/EntropyGenerator/EntropyGenerator.sol


### EntropyGenerator.initializeAlphaIndices() 

L6 - (contracts/EntropyGenerator/EntropyGenerator.sol#206-216) uses a weak PRNG: "slotIndexSelection = (hashValue % 258) + 512 (contracts/EntropyGenerator/EntropyGenerator.sol


### EntityTrading.buyNFT()

L7 - ❌ Reentrancy in EntityTrading.buyNFT(uint256) (contracts/EntityTrading/EntityTrading.sol:64-93):
	• transferToNukeFund(nukeFundContribution) (contracts/EntityTrading/EntityTrading.sol#75)
	• (success) = nukeFundAddress.call{value: amount}() (contracts/EntityTrading/EntityTrading.sol#115)
	• (success) = address(listing.seller).call{value: sellerProceeds}() (contracts/EntityTrading/EntityTrading.sol#78-80)
	• nftContract.transferFrom(address(this),msg.sender,tokenId) (contracts/EntityTrading/EntityTrading.sol#82)
	• transferToNukeFund(nukeFundContribution) (contracts/EntityTrading/EntityTrading.sol#75)
	• (success) = nukeFundAddress.call{value: amount}() (contracts/EntityTrading/EntityTrading.sol#115)
	• (success) = address(listing.seller).call{value: sellerProceeds}() (contracts/EntityTrading/EntityTrading.sol#78-80)
	• delete listings[listedTokenIds[tokenId]] (contracts/EntityTrading/EntityTrading.sol#84)
	• EntityTrading.listings (contracts/EntityTrading/EntityTrading.sol#20)

### DevFund.safeRewardTransfer

L8 - DevFund.safeRewardTransfer(address,uint256) (contracts/DevFund/DevFund.sol#88-98) sends eth to arbitrary user

Dangerous calls:

- (success) = address(to).call{value: amount}() (contracts/DevFund/DevFund.sol#94)


### NukeFund.nuke


L9 - NukeFund.nuke(uint256) (contracts/NukeFund/NukeFund.sol#157-190) sends eth to arbitrary user

Dangerous calls:

- (success) = address(msg.sender).call{value: claimAmount}() (contracts/NukeFund/NukeFund.sol#185)

