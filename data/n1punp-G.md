# 1. No-op modulo can be removed.

Source: 
- https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L53-L55
- https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L73-L75
- https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L91-L93

The above code snippets perform the following:
```solidity
uint256 pseudoRandomValue = uint256(
  keccak256(abi.encodePacked(block.number, i))
) % uint256(10) ** 78;
```
However, `uint256` has max value of `2^256-1` (or ~1.16 * 10^77, which is already smaller than `uint256(10)**78`. This means the modulo will not do anything, and thus can be removed without any logic changes.

Recommendation: Remove unnecessary `... % uint256(10) ** 78`

# 2. Can change `>=` to `==` to both be more gas efficient & logically more correct.

Source: https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L105-L114

The code snippet aims to create wrapped-around values (0-11 for `currentNumberIndex` and 0-769 for `currentSlotIndex`). To do this, it checks if the current value is greater than or equal to the threshold then sets it back to 09 (similar to modulo-addition). The current checks perform comparison via `>=` operator, which slightly consumes more gas than `==` operator. 

And since it is clear that each index only increments by 1, then it will always reach the equality first. Hence, we can replace `>=` with `==` without having any side-effects.

Recommendation: change both checks in [L105](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L105) and [L107](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L107) from `>=` to `==` .

