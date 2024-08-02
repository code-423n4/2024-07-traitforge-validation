### The variable `amountMinted` within the function `TraitForgeNft::mintWithBudget` is useless.

### Description
The amount of tokenIds minted is already tracked by the `_tokenIds` which is incremented within the `_mintInternal` function
```solidity
function mintWithBudget(bytes32[] calldata proof) ... {
    ...
    uint256 amountMinted = 0;  /// @audit useless. remove to save gas

    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
      _mintInternal(msg.sender, mintPrice);
      amountMinted++; /// @audit useless. remove to save gas
      ...
}
```

### Recommendation
Remove the `amountMinted` variable to save gas, as `_tokenIds` already tracks the number of minted tokens.