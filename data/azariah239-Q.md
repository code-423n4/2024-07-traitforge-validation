# Issue 1
## Summary

Several state variables are declared as `uint256` but will never utilize the full 2^256 range, leading to inefficient storage usage.

## Vulnerability Details

Multiple state variables in the contract are declared as `uint256` despite having practical upper limits far below the maximum value of this type. Specifically:

- `maxTokensPerGen`
- `startPrice`
- `currentGeneration`
- `maxGeneration`

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L22

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L33

These variables could be declared with smaller uint types to optimize storage and prevent potential misuse.

## Impact

The current implementation has two main impacts:

1. Gas Inefficiency: Unnecessarily large variable types consume more storage than needed, increasing gas costs for deployment and state-changing operations.

2. Safety Concerns: Using `uint256` for variables with much lower practical limits could allow setting these to unreasonably high values by mistake, potentially causing unexpected behavior or overflow in calculations involving these variables.


## Proof of Concept

N/A

## Tools used

Manual Auditing

# Issue 2

## Summary
The entropy calculation in phase 3 is not checked for `999999` value.

## Detailed description
The entropy calculation in phase 3 function `writeEntropyBatch3` does not check whether the generated random entropy value is `999999` which denotes the entity with the highest power. This can lead to various entities having highest power, in addition to those indicated by `slotIndexSelectionPoint` and `numberIndexSelectionPoint`.

There is a very little chance that this can happen and hence it is a low risk finding.

## Tools used
Manual Auditing

## LOC
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L84