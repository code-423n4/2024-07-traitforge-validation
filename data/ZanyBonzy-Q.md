
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

***

### 8. Excess tokens not refunded when lisitng for forging

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L125-L127

#### Impact

`forgeWithListed` accpts msg.value greater than or equal to the `forgingFee` but no attempt is made to refund excess to the caller, which can lead to loss of funds.
 
```solidity
  function forgeWithListed(
    uint256 forgerTokenId,
    uint256 mergerTokenId
  ) external payable whenNotPaused nonReentrant returns (uint256) {
//...
    uint256 forgingFee = _forgerListingInfo.fee;
    require(msg.value >= forgingFee, 'Insufficient fee for forging');
//...
    return newTokenId;
  }
```

#### Recommended Mitigation Steps
Recommend introducing a refund mechanism like the type in EntityTrading.sol

***

### 9. Functions querying `getEntropy` risk an array out of bound error.

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L168

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L102

#### Impact

`getEntropy` is called in three places, [`getNextEntropy`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L103), [`getPublicEntropy`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L127), and [`deriveTokenParameters`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L149). The function performs a check, ensuring that the `slotIndex` is <= `maxSlotIndex`. This check means it allows `slotIndex` from 0 to 770 rather than 769 as is to be expected when dealing with arrays.

```solidity
  function getEntropy(
    uint256 slotIndex,
    uint256 numberIndex
  ) private view returns (uint256) {
    require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');
//...
  }
```
This mostly affects `getPublicEntropy` and `deriveTokenParameters` which are view functions, and could have been a big deal in `getNextEntropy` if had it not been for this [check](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L114-L114).

Since the [`entropySlots`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L10) is expected to have just 770 elements, any user or external integration attempting to view the "last" entropy or token parameters risk encountering an out of bounds error.

```solidity
uint256[770] private entropySlots; 
```
#### Recommended Mitigation Steps

Recommend switching the to the < operator instead. The same can be done in [`getNextEntropy`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L102) function too.

***

### 10. Typographical error in EntropyGenerator.sol's constructor. It's traitForge and not traitforget

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L30-L33

#### Impact

In EntropyGenerator.sol, the allowed caller is spelt as `_traitForgetNft`
```solidity
  constructor(address _traitForgetNft) {
    allowedCaller = _traitForgetNft;
    initializeAlphaIndices();
  }
```

#### Recommended Mitigation Steps

It should be `_traitForgeNft`
***

### 11. Lack of `getTokenAge` function unlike declaration in the documentation.

Links to affected code *

https://github.com/TraitForge/GitBook/blob/main/GamePlay/Aging.md#2-function-gettokenageuint256-tokenid

#### Impact

The documentation declares a `getTokenAge` function with which to calculate a token's age in seconds. This function is however missing in the protocol's implementation.

#### Recommended Mitigation Steps
Recommend introducing this function.
***
### 11. Weird operator approval logic in `nuke` function

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L153-L162

#### Impact

In the `nuke` function, there are two approval checks. The first is standard, ensures that `msg.sender` is either the owner or is approved to spend the token. This means the token owner, token operator and any approved user can nuke the token. The next approval check then ensures that NukeFund.sol is approved to spend the token or is the operator of `msg.sender`. 
This logic means that NukeFund.sol has to be made the operator of the caller, not the owner for nuking to be successful. If owner approves NukeFund.sol to be one of his operators through the `setApprovalForAll` function. This operator status is ignored when the owner's approved user or another operator attempts to nuke the token. It fails due to the check marked @note below.

```solidity
  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {
    require(
      nftContract.isApprovedOrOwner(msg.sender, tokenId),
      'ERC721: caller is not token owner or approved'
    );
    require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)), //@note
      'Contract must be approved to transfer the NFT.'
    );
```

To prove this, the test case below can be copied and pasted into NukeFund.test.ts. 

```ts
  it('should nuke a token from operator', async function () {
    const tokenId = 1;

    // Mint a token
    await nft.connect(owner).mintToken(merkleInfo.whitelist[0].proof, {
      value: ethers.parseEther('1'),
    });

    // Send some funds to the contract
    await user1.sendTransaction({
      to: await nukeFund.getAddress(),
      value: ethers.parseEther('1'),
    });

    const prevNukeFundBal = await nukeFund.getFundBalance();
    // Ensure the token can be nuked
    expect(await nukeFund.canTokenBeNuked(tokenId)).to.be.true;

    const prevUserEthBalance = await ethers.provider.getBalance(
      await owner.getAddress()
    );
    
    // Approve user1
    await nft.connect(owner).approve(user1, tokenId);

    //user1 approves contract to be his own operator
    await nft.connect(owner).setApprovalForAll(await nukeFund.getAddress(), true);

    await nukeFund.connect(user1).nuke(tokenId);

    const curUserEthBalance = await ethers.provider.getBalance(
      await owner.getAddress()
    );

    const curNukeFundBal = await nukeFund.getFundBalance();
    expect(curUserEthBalance).to.be.gt(prevUserEthBalance);
    // Check if the token is burned
    // expect(await nft.ownerOf(tokenId)).to.equal(ethers.ZeroAddress);
    expect(await nft.balanceOf(owner)).to.eq(1);
    expect(curNukeFundBal).to.be.lt(prevNukeFundBal);
  });
```

The test reverts with a "Contract must be approved to transfer the NFT" error despite the fact that the owner already approved NukeFud.sol to be the operator.

#### Recommended Mitigation Steps

I believe this should check for owner's operator status instead.

```solidity
    require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(owner(), address(this)), //@note
      'Contract must be approved to transfer the NFT.'
    );
```
***


12.  `updateDev` can allow setting 0 weight to function as a `removeDev` function. It can help remove excess code

Links to affected code *

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L40-L49

#### Recommended Mitigation Steps
```solidity
  function updateDev(address user, uint256 weight) external onlyOwner {
    DevInfo storage info = devInfo[user];
   // require(weight > 0, 'Invalid weight');
    require(info.weight > 0, 'Not dev address');
    totalDevWeight = totalDevWeight - info.weight + weight;
    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;
    info.rewardDebt = totalRewardDebt;
    info.weight = weight;
    emit UpdateDev(user, weight);
  }
```

***

13.  Consider introducing a function to set the TraitForgeNFT's token/base uri