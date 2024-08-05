# L-1 EntropyGenerator::deriveTokenParameters() has forgePotential calculated incorrectly

## Impact

The `EntropyGenerator::deriveTokenParameters()` function calculates `forgePotential` incorrectly by using the first digit of the entropy value instead of the fifth digit, as mentioned in the documentation.

Refer to impacted line(s) of code here:

- [EntropyGenerator.sol#L136-L161](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L136-L161)

- [EntropyGenerator.sol#L198-L203](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L198-L203)

```text
Mentioned in DOCS:
Entropy: 123456
Entropy[4] = colour1 && forgePotential
```

```solidity
  function deriveTokenParameters(
    uint256 slotIndex,
    uint256 numberIndex
  )
    public
    view
    returns (
      uint256 nukeFactor,
      uint256 forgePotential,
      uint256 performanceFactor,
      bool isForger
    )
  {
    uint256 entropy = getEntropy(slotIndex, numberIndex);

    // example calcualtions using entropyto derive game-related parameters
    nukeFactor = entropy / 4000000;
    // @audit- docs mention using 5th digit as forge potential but uses the 1st digit instead
 @> forgePotential = getFirstDigit(entropy);
    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
  }
```

```
// @audit - returns first digit
  function getFirstDigit(uint256 number) private pure returns (uint256) {
    while (number >= 10) {
      number /= 10;
    }
    return number;
  }
```

## Tools Used

Manual Review

## Recommended Mitigation Steps

- Create a function that return 5th digit, and use it instead of `getFirstDigit()` in `deriveTokenParameters()` to calculate the `forgePotential`

```diff
// @audit - returns first digit instead of fifth
+ function getFifthDigit(uint256 number) private pure returns (uint256) {
+    // Remove the last four digits
+    uint256 fifthDigit = number / 10000;
+    // Return the last digit of the resulting number
+    return fifthDigit % 10;
+ }
```

```diff
  function deriveTokenParameters(
    uint256 slotIndex,
    uint256 numberIndex
  )
    public
    view
    returns (
      uint256 nukeFactor,
      uint256 forgePotential,
      uint256 performanceFactor,
      bool isForger
    )
  {
    uint256 entropy = getEntropy(slotIndex, numberIndex);

    // example calcualtions using entropyto derive game-related parameters
    nukeFactor = entropy / 4000000;
    // @audit- docs mention using 5th digit as forge potential but uses the 1st digit instead
- forgePotential = getFirstDigit(entropy);
+ forgePotential = getFifthDigit(entropy);

    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
  }
```