## Title:
Incorrect Boundary Check in Entropy Generation Logic

## Description:
The boundary check for currentSlotIndex in the getNextEntropy function incorrectly allows currentSlotIndex to be equal to maxSlotIndex, potentially leading to out-of-bounds errors. The check should ensure currentSlotIndex is strictly less than maxSlotIndex to prevent accessing an invalid index.
```
    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
```

## Impact:
The incorrect boundary check can lead to attempts to access an invalid index in the entropySlots array, potentially causing out-of-bounds errors. This can result in runtime exceptions, contract malfunctions, and potential security vulnerabilities if not properly handled.

## Recommendation:
Change the condition to ensure currentSlotIndex is strictly less than maxSlotIndex:
```
require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
```