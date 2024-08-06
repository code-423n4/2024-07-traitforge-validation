Division Before Multiplication

Description:
Performing division before multiplication can lead to precision issues in integer arithmetic and suboptimal gas usage.

Poc:

uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000;

https://github.com/code-423n4/2024-07-traitforge/blame/main/contracts/EntropyGenerator/EntropyGenerator.sol#L181

Recommendation:
Perform multiplication before division to maintain precision and optimize gas usage.

uint256 entropy = ((slotValue * (10 ** position)) / 10 ** 72) % 1000000;
