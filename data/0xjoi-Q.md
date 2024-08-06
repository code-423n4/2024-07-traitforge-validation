Dangerous strict equalities

❌ EntropyGenerator.deriveTokenParameters(uint256,uint256) (contracts/EntropyGenerator/EntropyGenerator.sol:136-161) 
uses a dangerous strict equality:    

```isForger = role == 0``` 

(contracts/EntropyGenerator/EntropyGenerator.sol#158)Recommendation: Don't use strict equality to determine if an account has enough Ether or tokens.

code link -> https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L158C5-L158C13