### 1. Title 
TYPO

### Severity
Low

## Description
In some inline comments and bug reports, typographical errors were found. In particular  

- [Line 37 of EntityTrading](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L37) says "function to lsit NFT for sale" but there should be "function to list NFT for sale".

- [Line 100 of EntityTrading](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L100) says "Only the seller can canel the listing." but there should be "Only the seller can cancel the listing.".

- [Line 184 of EntropyGenerator](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L184C44-L184C64) says "return the caculated entropy value" but there should be "return the calculated entropy value".

- [Line 180 of EntropyGenerator](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L180) says "slice the required [art of the entropy value" but there should be "slice the required part of the entropy value".
## Recommendations
Consider fixing these errors before deploying the contracts.

---

### 2. Title 
Underline for better understanding 
### Severity
Info

## Description
The `NukeFund.sol` and `TraitForgeNft.sol` contracts have variables `MAX_DENOMINATOR` and `maxTokensPerGen` respectively.
```solidity 
uint256 public constant MAX_DENOMINATOR = 100000;
```

```solidity 
uint256 public maxTokensPerGen = 10000;
```
This use of numbers makes it difficult to understand for developers and auditors 
## Recommendations
You should use underscores. For example:

```solidity 
uint256 public constant MAX_DENOMINATOR = 100_000;
```

```solidity 
uint256 public maxTokensPerGen = 10_000;
```
This will improve the clarity of the code 