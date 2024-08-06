# [L-01] When this `listingCount` accumulates too large, `fetchListings()` throws an out of gas error

## Impact
1. The `fetchListings()` function returns all the listed forge NFT information by iterating through the `listingCount` to obtain the array index. However, if the `listingCount` becomes too large (possibly > 3000 index), the array length exceeds the gas limit of the function, resulting in an out of gas error. See [example](https://ethereum.stackexchange.com/questions/135121/searching-lower-value-into-array-returns-out-of-gas-before-complete-the-total)


https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L15

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L21

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L48-L53

```
  function fetchListings() external view returns (Listing[] memory _listings) {
    _listings = new Listing[](listingCount + 1);
    for (uint256 i = 1; i <= listingCount; ++i) {
      _listings[i] = listings[i];
    }
  }
```
## Recommended Mitigation Steps
Since all variables are public, the work of the `fetchListings()` function in the contract can be done off-chain by directly calling `listingCount` and `mapping(uint256 => Listing) public listings;` using web3.js/ethers.js libraries

Example FetchListings.ts:
```
async function fetchListings() {
  const provider = new ethers.providers.JsonRpcProvider('https://mainnet.infura.io/v3/[YOUR_PROJECT_ID]');
  const contract = new ethers.Contract(contractAddress, contractABI, provider);

  const listingCount = await contract.listingCount();
  const listings = new Array(listingCount + 1);

  for (let i = 1; i <= listingCount; i++) {
    const listing = await contract.listings(i);
    listings[i] = {
      account: listing.account,
      tokenId: listing.tokenId,
      isListed: listing.isListed,
      fee: listing.fee
    };
  }

  return listings.slice(1);
}

fetchListings().then(listings => {
  listings.forEach(listing => {
    console.log(listing);
  });
});
```

# [L-02] The `_cancelListingForForging(uint256 tokenId)` function does not effectively delete listing data, resulting in `fetchListings()` returning a large number of invalid listing entries


## Impact
Notably, the delete listings[listedTokenIds[tokenId]]; operation does not actually remove any parameters at that index in the listings array; it merely resets the parameters to zero

**Such as:**

`listings[0] = Listing(msg.sender, 10, true, 1 ether);`

`listings[1] = Listing(msg.sender, 20, true, 2 ether);`

`listings[2] = Listing(msg.sender, 30, true, 3 ether);`

deleted listings:


`listings[0] = Listing({account: address(0), tokenId: 0, isListed: false, fee: 0});`

`listings[1] = Listing({account: address(0), tokenId: 0, isListed: false, fee: 0});`

`listings[2] = Listing({account: address(0), tokenId: 0, isListed: false, fee: 0});`


https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L48-L53

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L193-L197

```
  function _cancelListingForForging(uint256 tokenId) internal {
    delete listings[listedTokenIds[tokenId]];
    emit CancelledListingForForging(tokenId); // Emitting with 0 fee to denote cancellation
  }
```

## Recommended Mitigation Steps
Replace `delete` with `pop` and decrease the listingCount by using listingCount--

See [example1](https://stackoverflow.com/questions/71268574/how-to-delete-item-of-array-in-solidity)  [example2](https://docs.soliditylang.org/en/v0.8.20/types.html#data-location-and-assignment-behaviour)
