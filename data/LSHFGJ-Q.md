## Findings Summary

| Label | Description |
| - | - |
|L-01|Invalid `999999` entropy check|
|L-02|Lack max generation check|
|L-03|Unset `nukeFundAddress` may cause fund loss|
|L-04|Lack of refund procedure may cause fund loss|
|N-01|Unnecessary function `getPublicEntropy()`|
|N-02|Possible index out of bound|

## [L-01] Invalid `999999` entropy check
In function `writeEntropyBatch1()` and `writeEntropyBatch2()`, the check `pseudoRandomValue != 999999` is invalid. `pseudoRandomValue` here is a number expected to be 78-digit and far greater than 999999. The exact `999999` that should be checkd is the `entropy` sliced from `pseudoRandomValue`. If `999999` is returned by `getEntropy()`, it should be checked in `getNextEntropy()` and regenerated.
```diff
    function writeEntropyBatch1() public {
        require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');

        uint256 endIndex = lastInitializedIndex + batchSize1;
        unchecked {
        for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
            uint256 pseudoRandomValue = uint256(
            keccak256(abi.encodePacked(block.number, i))
            ) % uint256(10) ** 78;
            
-           require(pseudoRandomValue != 999999, 'Invalid value, retry.');
            entropySlots[i] = pseudoRandomValue;
        }
        }
        lastInitializedIndex = endIndex;
    }

    function writeEntropyBatch2() public {
        require(
        lastInitializedIndex >= batchSize1 && lastInitializedIndex < batchSize2,
        'Batch 2 not ready or already initialized.'
        );

        uint256 endIndex = lastInitializedIndex + batchSize1;
        unchecked {
        for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
            uint256 pseudoRandomValue = uint256(
            keccak256(abi.encodePacked(block.number, i))
            ) % uint256(10) ** 78;
-           require(pseudoRandomValue != 999999, 'Invalid value, retry.');
            entropySlots[i] = pseudoRandomValue;
        }
        }
        lastInitializedIndex = endIndex;
    }
```

## [L-02] Lack max generation check
The generation of Entity can be over preset 10.
```diff
    function _incrementGeneration() private {
        require(
            generationMintCounts[currentGeneration] >= maxTokensPerGen,
            'Generation limit not yet reached'
        );
        currentGeneration++;
+       require(currentGeneration <= maxGeneration, "can't be over max generation");
        generationMintCounts[currentGeneration] = 0;
        priceIncrement = priceIncrement + priceIncrementByGen;
        entropyGenerator.initializeAlphaIndices();
        emit GenerationIncremented(currentGeneration);
    }
```

## [L-03] Unset `nukeFundAddress` may cause fund loss
Fund may be transferred to zero address if `nukeFundAddress` not set.
```solidity
function forgeWithListed(
    uint256 forgerTokenId,
    uint256 mergerTokenId
) external payable whenNotPaused nonReentrant returns (uint256) {
    ...
    (bool success, ) = nukeFundAddress.call{ value: devFee }('');
    require(success, 'Failed to send to NukeFund');
    ...
}
```
Should be replaced by `transferToNukeFund()` in `EntityTrading.sol`:
```solidity
function transferToNukeFund(uint256 amount) private {
    require(nukeFundAddress != address(0), 'NukeFund address not set');
    (bool success, ) = nukeFundAddress.call{ value: amount }('');
    require(success, 'Failed to send Ether to NukeFund');
    emit NukeFundContribution(address(this), amount);
}
```

## [L-04] Lack of refund procedure may cause fund loss
If `msg.value > forgingFee`, the surplus will be lost in the contract.
```solidity
  function forgeWithListed(
    uint256 forgerTokenId,
    uint256 mergerTokenId
  ) external payable whenNotPaused nonReentrant returns (uint256) {
    ...
    uint256 forgingFee = _forgerListingInfo.fee;
    require(msg.value >= forgingFee, 'Insufficient fee for forging');
    ...
    uint256 devFee = forgingFee / taxCut;
    uint256 forgerShare = forgingFee - devFee;
    ...
    (bool success, ) = nukeFundAddress.call{ value: devFee }('');
    require(success, 'Failed to send to NukeFund');
    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');
    require(success_forge, 'Failed to send to Forge Owner');
    ...
  }
```

## [N-01] Unnecessary function `getPublicEntropy()`
`getEntropy()` can be set `public` to discard `getPublicEntropy()`.
```solidity
function getPublicEntropy(
    uint256 slotIndex,
    uint256 numberIndex
) public view returns (uint256) {
    return getEntropy(slotIndex, numberIndex);
}

function getEntropy(
    uint256 slotIndex,
    uint256 numberIndex
) private view returns (uint256) {
...
}
```

## [N-02] Possible index out of bound
`maxSlotIndex = 770` out of bound.
```diff
function getPublicEntropy(
    uint256 slotIndex,
    uint256 numberIndex
) public view returns (uint256) {
    return getEntropy(slotIndex, numberIndex);
}

function getEntropy(
    uint256 slotIndex,
    uint256 numberIndex
) private view returns (uint256) {
-   require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');
+   require(slotIndex < maxSlotIndex, 'Slot index out of bounds.');
...
}
```