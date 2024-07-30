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