https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L188

The function does not handle the case when number is 0. It will return 0 digits instead of 1.

fix like: 
if (number == 0) return 1;