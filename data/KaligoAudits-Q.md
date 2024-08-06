## Title:
[C-01] NFT contract can call `external` function `cancelListingForForging`, leads to removing any NFT, which is listed for forging

## Vulnerability Details:
The function `cancelListingForForging` is used for removing an NFT from the `listings` mapping, after it was used for forging. However, the function is marked as `external`, which means it can be called from outside contracts and EOAs. In that case, the `TraitForgeNft` contract can call this function and remove any NFT from the `listings` array, stopping an NFT from being forged with another NFT:

``` javascript
    function cancelListingForForging(uint256 tokenId) external whenNotPaused nonReentrant {
        require(
            nftContract.ownerOf(tokenId) == msg.sender || msg.sender == address(nftContract),
            "Caller must own the token"
        );
        require(listings[listedTokenIds[tokenId]].isListed, "Token not listed for forging");

        _cancelListingForForging(tokenId);
    }
```

In the first `require` statement, the function checks if the owner of the token id is `msg.sender` or the `TraitForgeNft` contract. In that case, the `TraitForgeNft` contract can call this function and remove any NFT from the `listings` array, stopping an NFT from being forged with another NFT.

## Impact
Centralization risk in `cancelListingForForging` for being marked as `external`, NFTs can be removed at any moment and not be able to forge.

## Recommended Mitigation
There are a few solutions to this problem:
1. Consider making the `cancelListingForForging` function as `internal`, so it can be called from another function:

``` diff
-   function cancelListingForForging(uint256 tokenId) external whenNotPaused nonReentrant {
+   function cancelListingForForging(uint256 tokenId) internal whenNotPaused nonReentrant {
        require(
            nftContract.ownerOf(tokenId) == msg.sender || msg.sender == address(nftContract),
            "Caller must own the token"
        );
        require(listings[listedTokenIds[tokenId]].isListed, "Token not listed for forging");

        _cancelListingForForging(tokenId);
    }
```

2. If for some reason this function has to be `external`, because the owner has to call it in some specific cases, you can put the `onlyOwner` modifier on the function:

``` diff
-   function cancelListingForForging(uint256 tokenId) external whenNotPaused nonReentrant {
+   function cancelListingForForging(uint256 tokenId) external whenNotPaused nonReentrant onlyOwner {
        require(
            nftContract.ownerOf(tokenId) == msg.sender || msg.sender == address(nftContract),
            "Caller must own the token"
        );
        require(listings[listedTokenIds[tokenId]].isListed, "Token not listed for forging");

        _cancelListingForForging(tokenId);
    }
```