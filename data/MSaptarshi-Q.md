# [QA-1] Contract inherits from `Pausable` but does not expose pausing/unpausing functionality
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L6
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L6
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L7
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L5
The TraitForge smart contracts inherits from OpenZeppelin's Pausable contract, but the _pause and _unpause methods are not exposed externally to be callable This shows that Pausable was used incorrectly and is possible to give out a false sense of security when actually contract is not pausable at all.

## Recommendations
expose the _pause and _unpause methods externally with access control.