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
Potential for Unused Funds
##Impact: Funds could be locked if not all devs claim.
##Proof of Concept:
No mechanism to sweep unclaimed funds.
##Mitigation: Implement a sweep function with a time lock.
```solidity
256 public constant CLAIM_PERIOD = 365 days;
uint256 public lastFundingTime;

function sweepUnclaimedFunds() external onlyOwner {
    require(block.timestamp > lastFundingTime + CLAIM_PERIOD, "Claim period not over");
    uint256 balance = address(this).balance;
    (bool success, ) = owner().call{value: balance}("");
    require(success, "Transfer failed");
}
```