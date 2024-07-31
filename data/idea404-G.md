This issue contains Gas Optimisation suggestions for the `DevFund.sol` file: 

1. Store totalDevWeight in a local variable during the receive function to minimize state variable access:

```solidity
  receive() external payable {
    if (totalDevWeight > 0) {
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

Original Contract Gas Used: 41737
Optimized Contract Gas Used: 36153
Gas Savings: 5584

Test used: 

```ts
    const originalTx = await user2.sendTransaction({
      to: await devFund.getAddress(),
      value: ethers.parseEther('1'),
      gasLimit: 200000
    });
    const originalReceipt = await ethers.provider.getTransactionReceipt(originalTx.hash);
    const originalGasUsed = Number(originalReceipt?.gasUsed);
    console.log("Original Contract Gas Used:", originalGasUsed);

    const optimizedTx = await user2.sendTransaction({
      to: await optimizedDevFund.getAddress(),
      value: ethers.parseEther('1'),
      gasLimit: 200000
    });
    const optimizedReceipt = await ethers.provider.getTransactionReceipt(optimizedTx.hash);
    const optimizedGasUsed = Number(optimizedReceipt?.gasUsed);
    console.log("Optimized Contract Gas Used:", optimizedGasUsed);

    const gasSavings = originalGasUsed - optimizedGasUsed;
    console.log("Gas Savings:", gasSavings);
  });
```

2. Use local variables in `removeDev`

```solidity
    function removeDev(address user) external onlyOwner {
        DevInfo storage info = devInfo[user];
        require(info.weight > 0, 'Not dev address');
        
        uint256 oldWeight = info.weight;
        uint256 newTotalDevWeight = totalDevWeight - oldWeight;
        
        info.pendingRewards += (totalRewardDebt - info.rewardDebt) * oldWeight;
        info.rewardDebt = totalRewardDebt;
        info.weight = 0;
        totalDevWeight = newTotalDevWeight;
        
        emit RemoveDev(user);
    }
```

Original Contract Gas Used for removeDev: 37622
Optimized Contract Gas Used for removeDev: 33862
Gas Savings for removeDev: 3760

test used:

```ts
  it('should compare gas usage of removeDev function', async () => {
    const originalTx = await devFund.removeDev(user1.address, { gasLimit: 200000 });
    const originalReceipt = await ethers.provider.getTransactionReceipt(originalTx.hash);
    const originalGasUsed = Number(originalReceipt?.gasUsed);
    console.log("Original Contract Gas Used for removeDev:", originalGasUsed);

    const optimizedTx = await optimizedDevFund.removeDev(user1.address, { gasLimit: 200000 });
    const optimizedReceipt = await ethers.provider.getTransactionReceipt(optimizedTx.hash);
    const optimizedGasUsed = Number(optimizedReceipt?.gasUsed);
    console.log("Optimized Contract Gas Used for removeDev:", optimizedGasUsed);

    const gasSavings = originalGasUsed - optimizedGasUsed;
    console.log("Gas Savings for removeDev:", gasSavings);
  });
```
