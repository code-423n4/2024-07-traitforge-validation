## [Gas] Unnecessary trasaction successful and emit events, costs unnecessary gas. 

**Description:** In `EntropyGenerator::getNextEntropy` trasaction executed on 770th entropy position but the position starts from `currentSlotIndex = 0` to `769` total 770 entropies but the `getNextEntropy` function executes call another function `getEntropy` which also allows 770th slot and then return an entropy and further `getNextEntropy` function executes and emit an event. 

**Impact:** Unncessary gas costs for unwanted trasaction.

**Proof of Concept:**
1. Address caller calls `getNextEntropy` with slotIndex = 770.
2. this function calls another `getEntropy` function. Most probably it returns 0.
3. After this, `getNextEntropy` function emits an event and return entropy.

**Recommended Mitigation:** Make following changes in `EntropyGenerator::getNextEntropy` and `EntropyGenerator::getEntropy` : 
```diff
  function getNextEntropy() public onlyAllowedCaller returns (uint256) {
  
-    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
+    require(currentSlotIndex < maxSlotIndex, 'Max slot index reached.');
```

```diff
  function getEntropy(
    uint256 slotIndex,
    uint256 numberIndex
  ) private view returns (uint256) {
-    require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');
+    require(slotIndex < maxSlotIndex, 'Slot index out of bounds.');
```