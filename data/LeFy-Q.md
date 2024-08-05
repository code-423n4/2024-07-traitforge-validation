Unnecessary Calculations in entorpy calculations

```Solidity
 function writeEntropyBatch1() public {
    require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');

    uint256 endIndex = lastInitializedIndex + batchSize1; // calculate the end index for the batch
    unchecked {
      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
        uint256 pseudoRandomValue = uint256(
          keccak256(abi.encodePacked(block.number, i))
        ) % uint256(10) ** 78; // generate a  pseudo-random value using block number and index
        require(pseudoRandomValue != 999999, 'Invalid value, retry.');
        entropySlots[i] = pseudoRandomValue; // store the value in the slots array
      }
    }
    lastInitializedIndex = endIndex;
  }

```

Firstly uint256(10) ** 78 is greater than uint256 max value, so any uint256 value % uint256(10) ** 78 will always be same as the uint256 value itself. So no need for % operation and also unchecked block.