
### 1. `nukeFundAddress` should be set in the constructor in multiple contracts to avoid risk of first set of funds sent to it being lost

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L156

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L361

#### Impact

In TraitForgeNft.sol and EntityForging.sol, the `nukeFundAddress` is not set in constructor, therefore, within a brief period betwen the contract deployment and the functions to set the address, Attempts to [`forge`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L156) and/or [mint](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L358) risks sending the tokens to address 0 causing loss of funds. The sending will succeed because low level .call will return success even after querying a non existent address.

#### Recommended Mitigation Steps

Recommend setting `nukeFundAddress` or checking for address 0 like is done in [EntityTrading.sol](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L113)
***

### 2. `canTokenBeNuked` problematically queries `tokenAgeInSeconds` against its `getTokenLastTransferredTimestamp`

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L190-L193

#### Impact

To see if a token can be nuked, the `canTokenBeNuked` function is called. The function, rather than query the token's creation timestamp (`getTokenCreationTimestamp`), checks for the last transferred timestamp instead. As a result, a token that is ready for nuking will have its status revoked if transferred. On one hand, this might be to incetivze users to hold the tokens for longer, on the other hand, this might lead to longer wait periods for owners in certain scenarios.

```solidity
  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {
    // Ensure the token exists
    require(
      nftContract.ownerOf(tokenId) != address(0),
      'ERC721: operator query for nonexistent token'
    );
    uint256 tokenAgeInSeconds = block.timestamp -
      nftContract.getTokenLastTransferredTimestamp(tokenId);
    // Assuming tokenAgeInSeconds is the age of the token since it's holding the nft, check if it's over minimum days held
    return tokenAgeInSeconds >= minimumDaysHeld;
  }
```

For instance, a user has held a token past minimumDaysHeld, making it ready for nuking. He lists his token for sale at a price. This transfers the token from the user to EntityTrading.sol, thereby resetting his `LastTransferredTimestamp` and making his token no longer ready for nuking. If he cancels the sales listing by deciding to nuke, the EntityTrading contract transfers the token back to the user, resetting his `LastTransferredTimestamp` again. He now has to wait even longer before he can successfully nuke the token.

#### Recommended Mitigation Steps

Recommend checking for tokenCreationTime instead. 

```diff
  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {
//...
    uint256 tokenAgeInSeconds = block.timestamp -
-      nftContract.getTokenLastTransferredTimestamp(tokenId);
+      nftContract.getTokenCreationTimestamp(tokenId);

//...
  }
}
```
Or making a special exception for tokens listed in EntityTrading.sol during transfers.

```diff
  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 firstTokenId,
    uint256 batchSize
  ) internal virtual override {
    super._beforeTokenTransfer(from, to, firstTokenId, batchSize);

    uint listedId = entityForgingContract.getListedTokenIds(firstTokenId);
    /// @dev don't update the transferred timestamp if from and to address are same

-    if (from != to) {
-      lastTokenTransferredTimestamp[firstTokenId] = block.timestamp;
-    }

    if (listedId > 0) {
      IEntityForging.Listing memory listing = entityForgingContract.getListings(
        listedId
      );
      if (
        listing.tokenId == firstTokenId &&
        listing.account == from &&
        listing.isListed
      ) {
        entityForgingContract.cancelListingForForging(firstTokenId);
      }
    }

    require(!paused(), 'ERC721Pausable: token transfer while paused');
+    if (from == entityTradingContract ) {
          return;
+   } else if (to ==) {}
  
  
  else {
+      if (from != to) {
+      lastTokenTransferredTimestamp[firstTokenId] = block.timestamp;
+    }
+ }

  }
```
***


### 3. Unnecessary approval checks in various places

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L47-L54

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L153-L161

#### Impact

When listing tokens for sale, the token is attempted to be transferred from the lister to the EntityTrading contract. The `listNFTForSale` first checks if the EntityTrading contract is approved to tranfer the token by the token owner. This is unnecessary as the same check is repeated in the [`transferFrom`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/fd81a96f01cc42ef1c9a5399364968d0e07e9e90/contracts/token/ERC721/ERC721.sol#L152) function when transferring ERC721 tokens.

```solidity
    require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );

    nftContract.transferFrom(msg.sender, address(this), tokenId); 
```

The same can be observed in the `nuke` function in NukeFund.sol. The contract first checks if NukeFund.sol is approved to spend the token. But the [burn](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L143) function in TraitForgeNFT.sol also performs the same check. Making this unnecessary.

```solidity
  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {
    require(
      nftContract.isApprovedOrOwner(msg.sender, tokenId),
      'ERC721: caller is not token owner or approved'
    );
    require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );
//...
    nftContract.burn(tokenId); // Burn the token
//...
  }

```


#### Recommended Mitigation Steps

Recommend removing the unneeded code.
***

### 4. `claim` function can be refactored to call `pendingRewards` instead, since its first half performs the same functionality

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L61-L75

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L77-L81

#### Impact

In the first half of claim function in DevFund.sol, it can be observed that it gets the caller's `devInfo` then calculates his pending rewards.

```solidity
  function claim() external whenNotPaused nonReentrant {
    DevInfo storage info = devInfo[msg.sender];

    uint256 pending = info.pendingRewards +
      (totalRewardDebt - info.rewardDebt) *
      info.weight;
//...
  }
```

This part is also observed to be the same in the `pendingRewards` function.
```solidity
  function pendingRewards(address user) external view returns (uint256) {
    DevInfo storage info = devInfo[user];
    return
      info.pendingRewards + (totalRewardDebt - info.rewardDebt) * info.weight;
  }
```

A potential refactoring to allow for lighter code is to simply call the `pendingRewards` function in `claim` since they do the same thing.

#### Recommended Mitigation Steps

```diff
  function claim() external whenNotPaused nonReentrant {
-    DevInfo storage info = devInfo[msg.sender];

-    uint256 pending = info.pendingRewards +
-      (totalRewardDebt - info.rewardDebt) *
-      info.weight;
+    pendingRewards(msg.sender)
//...
  }
```
***


### 5. Users through sybils can forge token with their own forger and merger tokens.

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L111-L118

#### Impact

In `forgeWithListed` function, there's an invariant that a user cannot own both the merger and forger token. This can easily be bypassed by sybils. A user can transfer his forger to another address he controls, list the token for a forging, mint his actual address a merger token, then forge. The only potential cost is the [`devFee`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L146-L146) since the remaining fee is paid to the `seller` which is an address he controls. After this is done, he can always return his ETH and token to his main address.

```solidity
    require(
      nftContract.ownerOf(mergerTokenId) == msg.sender,
      'Caller must own the merger token'
    );
    require(
      nftContract.ownerOf(forgerTokenId) != msg.sender,
      'Caller should be different from forger token owner'
    );
```

#### Recommended Mitigation Steps

A potential mitigation might be to query the token's `initialOwner` and ensuring that its not the caller. Beyond this, there's almost no fix.

***


### 6. `claim` function should not be pausable

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L61

#### Impact

Regardless of ongoing situation, emergency or not, users/devs should be able to claim their rewards at any time they wish. Also, pausing the contract and renouncing admin role can lock the tokens in the protocol permanently.
```solidity
  function claim() external whenNotPaused nonReentrant {
//...
```
#### Recommended Mitigation Steps
Recommend removing the `whenNotPaused` modifier.
***

### 7. Unused check for generation count in `_incrementGeneration` function.

22. Redundant checks since functions taht call the function already performs the same check

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L345-L349

#### Impact

`_incrementGeneration` is called in two major places. In [`mintInternal`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L281-L283) and [`_mintNewEntity`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L331-L336). The condition for the function to be called in both cases is that `(generationMintCounts[currentGeneration] >= maxTokensPerGen)`. This same condition is then repeated in the function, making it redundant and un needed, as the function will not be called if the condition had not been fulfilled in the first place.

```solidity
  function _incrementGeneration() private {
    require(
      generationMintCounts[currentGeneration] >= maxTokensPerGen,
      'Generation limit not yet reached'
    );
```
#### Recommended Mitigation Steps

Recommend removing the check from `_incrementGeneration` function.