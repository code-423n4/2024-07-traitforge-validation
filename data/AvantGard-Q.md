
## [L-01] Disallow listing maxGeneration Entities (currently 10)

Since NFTs belonging to maxGeneration cannot forge further generations.
They should be restricted from listing for forging inside the `listForForging` function
until the `maxGeneration` is increased.
```
if (nftContract.getTokenGeneration(tokenId) == maxGeneration) {
    revert MaxGenCantList();
}
```


## [L-02] `fetchListing` should not fetch previously cancelled listings

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L48-L53
```
struct Listing {
    address account;
    uint256 tokenId;
    bool isListed;
    uint256 fee;
  }
```
Use the `isListed` bool to fetch only the active listings


## [L-03] `getListedTokenIds` should return 0 when queried with unlisted TokenIds
  
 `getListedTokenIds` is used by the  `_beforeTokenTransfer` hook 
 to get the `listingCount` of the tokenID.
 
 However `getListedTokenIds` is not updated everytime an entity is removed from listing.
 This provides false data when queried thru `getListedTokenIds` function.
	
 `listedTokenIds` mapping needs to be reset when a listing is cancelled.

## [L-04] `amountMinted` should be emitted as an event while `mintWithBudget` is called
```
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L217
    emit MintedWithBudget(
      msg.sender, 
	    amountMinted,
			budgetLeft 
    );
```

## [L-05] `writeEntropyBatch3` is missing the `pseudoRandomValue` check

`writeEntropyBatch1` &  `writeEntropyBatch2` functions have the following check to ensure
pseudoRandomValue doesnt equal 999999.

```
require(pseudoRandomValue != 999999, 'Invalid value, retry.');
```
But `writeEntropyBatch3` is missing this check.

## [NC-01] Use the term `NukeFund` instead of `devFee` for clarity

```
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L146

uint256 devFee = forgingFee / taxCut; (wrong)

uint256 NukeFund = forgingFee / taxCut; (Corrected)
```