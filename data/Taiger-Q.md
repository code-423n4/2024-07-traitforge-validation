1.Imbalanced distribution caused by excessive weight

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/DevFund/DevFund.sol#L35

The owner privileged role can add weight to any developer through the addDev method. Since the calculation of pending is based on weight (info.pendingRewards + (totalRewardDebt - info.rewardDebt) * info.weight), if the weight of a developer is too large, the reward he can receive at a certain moment will exceed the balance of the contract. Although the contract has a safeRewardTransfer function to prevent the transfer of an amount exceeding the balance, this means that other developers may not be able to receive the rewards they deserve in the future, or the rewards will gradually decrease.
