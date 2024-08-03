| Issue   | Headline                                               |
|---------|--------------------------------------------------------|
| QA-01 | [Numbers can be simplified](#qa-01-numbers-can-be-simplified) |
| QA-02 | [`listedTokenIds` are not deleted](#qa-02-listedtokenids-are-not-deleted) |
| QA-03 | [`_distributeFunds` inside the main function](#qa-03-_distributefunds-inside-the-main-function) |

### [QA-01] Numbers can be simplified
[maxNumberIndex](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L18) is set to 13 but is used only once, where it is subtracted by 1.

```solidity
18:    uint256 private maxNumberIndex = 13;

       ...
105:   if (currentNumberIndex >= maxNumberIndex - 1) {
106:       currentNumberIndex = 0;
```

Set it to 12 instead of subtracting 1.

### [QA-02] `listedTokenIds` are not deleted
[cancelListing](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L94) and [buyNFT](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L63) both delete `listings`, but they forget to delete `listedTokenIds`.

```solidity
        nftContract.transferFrom(address(this), msg.sender, tokenId); 

        //@audit QA - we don't delete `listedTokenIds`
        delete listings[listedTokenIds[tokenId]];
        emit ListingCanceled(tokenId, msg.sender);
```

This might be fine for now, but it can cause integration and front-end problems. It has the potential to cause future issues if the system is upgraded or expanded.

The same applies to EntityForging [_cancelListingForForging](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L193-L197), where `listedTokenIds` are not deleted.

Additionally, in [_beforeTokenTransfer](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L367-L395), unnecessary operations are performed. When it calls `getListedTokenIds`, it always receives an ID, even if the NFT is not listed anymore (because `listedTokenIds` is not deleted). This means it will always query `getListings` and check if the NFT is actually listed.

```solidity
    uint256 listedId = entityForgingContract.getListedTokenIds(firstTokenId);

    if (listedId > 0) {
        IEntityForging.Listing memory listing = entityForgingContract.getListings(listedId);

    if (listing.tokenId == firstTokenId && listing.account == from && listing.isListed) {
        entityForgingContract.cancelListingForForging(firstTokenId);
        }
```

### [QA-03] `_distributeFunds` inside the main function
[_distributeFunds](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L308) is called inside [_mintInternal](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L280-L309) to distribute the `msg.value` to the nuke fund.

```solidity
    emit Minted(msg.sender, newItemId, currentGeneration, entropyValue, mintPrice);

    _distributeFunds(mintPrice);
```

An improvement would be to move it from [_mintInternal](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L280-L309) to [mintToken](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L181-L200) and [mintWithBudget](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L202). Currently, if called from [mintWithBudget](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L202), the minting value of each NFT would be sent on every mint. If we mint 10 NFTs, that would result in 10 unnecessary external calls. Only one is needed.