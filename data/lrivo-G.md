# TraitForgeNft:onlyWhitelisted should use `verifyCalldata()` instead of `verify()`

Both `mintToken()` and `mintWithBudget()` store the proof array in calldata.

But, the `onlyWhitelisted` modifier uses `MerkleProof:verify()` which accepts a memory proof array, meaning it will copy the proof array from calldata into memory, consuming more gas due to memory expansion cost.

### Mitigation
```diff
modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
    if (block.timestamp <= whitelistEndTime) {
      require(
-       MerkleProof.verify(proof, rootHash, leaf),
+       MerkleProof.verifyCalldata(proof, rootHash, leaf),
        'Not whitelisted user'
      );
    }
    _;
  }
```