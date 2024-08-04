## Title: [L1] No check to ensure `budgetLeft` is greater than `mintPrice` initially in `mintWithBudget` function, might lead to silent failure of function 

## Summary:
In `mintWithBudget` function although they are checking if the mint price is great than the amount sent in the `while` loop.
```js
while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen){...}
```
But there's no check present initially to handle the situation where the user might transfer less funds than required. 

So, in that condition the function might silently fail as there's no revert statement or checks present to handle that situation, the `mintToken` function ensures this check.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L202-L225

## Recommended Mitigation:
Implement a check to ensure `budgetLeft` is greater than `mintPrice` initially in `mintWithBudget` function.
E.g.
```js
    require(msg.value >= mintPrice, 'Insufficient ETH send for minting.');
```

---

## Title: [QA] `pendingRewards` function is defined to calculate pending rewards but still other functions manually calculate rewards.

## Summary:
`pendingRewards` function is defined to calculate rewards, but in all the other functions of the `DevFund.sol` contract, where calculation of pending rewards is required, it is calculated manually.
E.g.
```
info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;
```
This is done in the following functions:
- `claim()`
- `removeDev()`
- `updateDev()`

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L77-L81

## Recommended Mitigation:
`pendingRewards()` function is calculating `pendingRewards` itself, it would be a better and modular approach to utilize this function in calculation of `pendingRewards` in the other function of this contract.