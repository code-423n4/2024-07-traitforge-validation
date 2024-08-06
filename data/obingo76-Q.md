Emit Events for All Critical State Changes
##Impact: Difficulty in tracking off-chain and reduced transparency.
##Proof of Concept:
No event emitted for ```totalRewardDebt``` updates.
##Mitigation:
 Add events for all critical state changes.
```solidity
TotalRewardDebtUpdated(uint256 newTotalRewardDebt);

receive() external payable {
    // ... existing code
    totalRewardDebt += amountPerWeight;
    emit TotalRewardDebtUpdated(totalRewardDebt);
    // ... rest of the function
}
```