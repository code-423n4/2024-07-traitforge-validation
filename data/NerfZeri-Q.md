The revert errors in `EntityTrading:buyNFT` are misleading as the function checks the listing price before the listing array.

the setup of the function is as follows:

```javascript
  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
    Listing memory listing = listings[listedTokenIds[tokenId]];
    require(
      msg.value == listing.price,
      'ETH sent does not match the listing price'
    );
    require(listing.seller != address(0), 'NFT is not listed for sale.');
```

tokens that have been sold and attempted to be purchased with the original listing price revert with the error `ETH sent does not match the listing price` which is misleading.

The contract sets the price of a sold NFT to 0 so the correct error only shows up when sending a value of 0, this means users who try to purchase an NFT which has been recently sold will think they have sent the wrong amount and try to buy it again, resulting in an unnecessary sped of gas and time for the user.

consider changing the order of operations to check the listing before the price to avoid this issue, as shown below:

```javascript
  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
    Listing memory listing = listings[listedTokenIds[tokenId]];
    require(listing.seller != address(0), 'NFT is not listed for sale.');
    require(msg.value == listing.price,'ETH sent does not match the listing price');
```