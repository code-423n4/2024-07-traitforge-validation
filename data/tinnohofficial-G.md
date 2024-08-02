### The variable `listing` in `EntityTrading::cancelListing` function can be changed from `storage` to `memory` for gas efficiency

### Description
Variable `listing` is only read (i.e ` listing.isActive ` ) and there no any write operations on it. Thus it is better to use `memory` instead.
```solidity
function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {
@>  Listing storage listing = listings[listedTokenIds[tokenId]];
    ...
    require(listing.isActive, 'Listing is not active.');
```

### Recommendation
Change line to `Listing memory listing = ...`

