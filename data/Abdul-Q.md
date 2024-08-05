# LOW FINDINGS IN DEVFUND.SOL

## [L-1] Updated weight should be different

### *Description:*
Function updateDev updates the weight of dev but not checking whether the given weight value is different.

### *Recommended Mitigation:*

```diff
 function updateDev(address user, uint256 weight) external onlyOwner {
     DevInfo storage info = devInfo[user];
-    require(weight > 0, 'Invalid weight');
+    require(weight > 0 && info.weight != weight, 'Invalid weight');
     require(info.weight > 0, 'Not dev address');
     totalDevWeight = totalDevWeight - info.weight + weight;
     info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;
     info.rewardDebt = totalRewardDebt;
     info.weight = weight;
     emit UpdateDev(user, weight);
   }
```

## [L-2] Implement functions to pause and unpause contract.

### *Description:*
Add functions to change the state of DevFund as pause and unpause.

## [L-3] Implement functions to change ownership of the DevFund contract

### *Description:* 
Add functions to change ownership of contract.

