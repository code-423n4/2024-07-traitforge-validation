1. Incorrect Token Limit Check in mintWithBudget Function

The issue in the `TraitForgeNft::mintWithBudget` function is due to a mistake in the while loop condition that checks if the maximum number of tokens per generation `maxTokensPerGen` has been reached. Instead of checking how many tokens have been minted in the current generation, the condition checks the `_tokenIds` value. Since `_tokenIds` can be much higher than `maxTokensPerGen`, this can cause the function to stop working properly when `_tokenIds` gets too high. As a result, it won't mint new tokens even if the current generation hasn't reached its limit yet.


```solidity
  function mintWithBudget(
    bytes32[] calldata proof
  )
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  {
    uint256 mintPrice = calculateMintPrice();
    uint256 amountMinted = 0;
    uint256 budgetLeft = msg.value;

    // @audit
    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
```
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L215