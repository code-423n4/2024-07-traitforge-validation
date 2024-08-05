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

## Title: [L2] Using `block.number` is bad source of randomness in function `initializeAlphaIndices`

## Impact
Miners can gain unfair advantage over normal users to get desired result.


## Proof of Concept
The `initializeAlphaIndices` function uses `block.number` which is a bad source to generate randomness.
```
  function initializeAlphaIndices() public whenNotPaused onlyOwner {
    uint256 hashValue = uint256(
      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))
      //@audit using block.number for randomness
    );

    uint256 slotIndexSelection = (hashValue % 258) + 512;
    uint256 numberIndexSelection = hashValue % 13;

    slotIndexSelectionPoint = slotIndexSelection;
    numberIndexSelectionPoint = numberIndexSelection;
  }
```


The above mentioned function uses the code below to generate `hashValue`.
```js
    uint256 hashValue = uint256(
      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))
    );
```
The function will return the same result for the same block.

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L206-L216)
This exact way of generating random number using `blockhash` , `block.number` and `block.timestamp` can be easily predicted by a miner. As shown [here](https://www.slowmist.com/articles/solidity-security/Common-Vulnerabilities-in-Solidity-Randomness.html)

Although this is marked as `onlyOwner` but the vulnerability still remains.

## Tools Used
Manual review

## Recommended Mitigation Steps
Use better sources to generate randomness like, [Chainlink Keepers](https://docs.chain.link/vrf).

---

## Title: [NC] `pendingRewards` function is defined to calculate pending rewards but still other functions manually calculate rewards.

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

## Tools used:
Manual Review

## Recommended Mitigation:
`pendingRewards()` function is calculating `pendingRewards` itself, it would be a better and modular approach to utilize this function in calculation of `pendingRewards` in the other function of this contract.

