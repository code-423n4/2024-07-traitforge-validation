# [L-01] `EntityForging::fetchListing` sets wrong length of returned array and returns listings already canceled
There are two problems with `fetchListing` implementation:
1. It sets the length of the array to `listingCount + 1`, and `listingCount` is never decremented when cancelling listings. So cancelled listing will be included in the return array (with values having default zero-value)
2. The array length is wrongly initialized:
    ```solidity
    function fetchListings() external view returns (Listing[] memory _listings) {
      _listings = new Listing[](listingCount + 1); // @audit no need +1
      for (uint256 i = 1; i <= listingCount; ++i) {
        _listings[i] = listings[i];
      }
    }
    ```
    Whenever there is a new listing, `listingCount` is [incremented before creating it](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L95-L97), so the first listing starts from one, not from zero:
    ```solidity
    // function: listForForging

    ++listingCount;
    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);
    listedTokenIds[tokenId] = listingCount;

    emit ListedForForging(tokenId, fee);
    ```
    So, the length of the returned array should be:
    ```solidity
    _listings = new Listing[](listingCount);
    ```