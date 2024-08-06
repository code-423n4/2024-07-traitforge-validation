### [L-1] `TraitForgeNft::isForge()` will always return true if NFT with tokenId is not minted yet (Root + Impact)

**Description**

The role indicator is determined by taking the modulo 3 of the token's entropy. This entropy value is stored in a mapping within the contract. If the NFT with the specified `tokenId` has not been minted yet, the entropy value in the mapping will be `0`. Since `0 % 3 = 0`, this condition will always evaluate to true.

**Impact**

This function, if used by the front-end or users, could provide incorrect information.

**Proof of Concepts**

```javascript
function isForger(uint256 tokenId) public view returns (bool) {
@>    uint256 entropy = tokenEntropy[tokenId]; 
@>    uint256 roleIndicator = entropy % 3;
      return roleIndicator == 0;
}
```

**Recommended mitigation**

```diff
function isForger(uint256 tokenId) public view returns (bool) {
+   require(_exists(tokenId), "ERC721: operator query for nonexistent token");
    uint256 entropy = tokenEntropy[tokenId]; //!If the token is not minted yet, it will always return true;
    uint256 roleIndicator = entropy % 3;
    return roleIndicator == 0;
}
```