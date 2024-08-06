## 1. Recommended to use `ERC721.safeTransferFrom` instead of `ERC721.transferFrom` for the NFT transfer in the `EntityTrading.buyNFT` function

In the `EntityTrading.buyNFT` function the bought `NFT` is transferred to the `buyer (msg.sender)` as shown below:

```solidity
    nftContract.transferFrom(address(this), msg.sender, tokenId);
```

As it is seen above the transfer happens by calling the `transferFrom` function. The issue is if the msg.sender is a contract which can not handle ERC721 then the transferred NFT could get locked in the `buyer contract` permenantly.

Hence it is recommended to use the `ERC721.safeTransferFrom` function inplace of the `transferFrom` function for the NFT transfer.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L81

## 2. `weight != info.weight` check is not implemented in the `DevFund.updateDev` function

The `DevFund.updateDev` function is used to update the `weight` of a `dev` in the `DevFund` contract. But there is no check in place to ensure that passed in `weight != info.weight`. Thus if the `updateDev` function is called with the same `weight` then transaction will still execute without any state change thus making a redundant transaction.

Hence it is recommended to implement input validation for `weight != info.weight` at the beginning of the `DevFund.updateDev` function.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/DevFund/DevFund.sol#L40-L43

## 3. The `DevFund.updateDev` function can be used to both update a dev and remove a dev from the `DevFund` contract

The `DevFund.updateDev` function is used to update the `weight` of a `dev` in the `DevFund` contract. The  `DevFund.removeDev` function is used to remove a `dev` from the `DevFund` contract by setting its info.weight = 0. 

But there is no need of two functions for this. The `DevFund.updateDev` function can be used to remove the `dev` from the `DevFund` contract by allowing weight == 0 to be passed in. This will further optimize the `DevFund` contract.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/DevFund/DevFund.sol#L40-L49
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/DevFund/DevFund.sol#L51-L59

## 4. If `TraitForgeNft.burn` is called directly it could be loss of funds to the owner or the approved address

The `TraitForgeNft.burn` function is used to allow the approved address or the owner of a `tokenId` to burn it. But the issue is this function is `external` and can be called by the `approved address or owner` at anytime. But the correct procedure to burn the `tokenId (entity)` in the `TraitForgeNft` protocol is by calling the `NukeFund.nuke` function. 

The `NukeFund.nuke` function will transfer the respective share of the `NukeFund` to the owner or the approved address (msg.sender) in addition to burning the `tokenId`. But if the `TraitForgeNft.burn` is directly called by mistake by the `owner or the approved address`, they will not get the respective share of the `NukeFund`. Hence this will be loss of funds to the owner or the approved address.

Hence it is recommended to ensure only the `NukeFund` contract can call the `TraitForgeNft.burn` function during the execution of the `nuke` transaction. This can be done by implementing the access control in the `TraitForgeNft.burn` function.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L141-L151
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L176-L178

## 5. The excessive ETH sent in the `TraitForgeNft.mintToken` transaction could be consumed befroe refunded to the `msg.sender`

The `TraitForgeNft.mintToken` function is used to mint the `TraitForgeNft` tokens to the `msg.sender` address, and the exceess value of the `msg.value` will be refunded to him. Now let's consider the following scenario:

1. The current price of the `TraitForgeNft` is 0.1 Eth.
2. The user A wants to mint this NFT and calls the `TraitForgeNft.mintToken` with `0.15 Eth` knowing he would get a refund of `0.05 Eth` (The user is not concerned about the excessive funds transferred to the transaction since he is getting the refund).
3. A user B also wants to mint `TraitForgeNfts` and calls the `TraitForgeNft.mintWithBudget` function by front-running the User A's transaction, where User B mints the required number of tokens in a loop.
4. Now the current price of the next NFT to be minted is `0.15 Eth`.
5. Now the `User A's` transaction is executed and he will mint the NFT for `0.15 Eth` and he will not get any refund since all his supplied funds are now used to mint the NFT.
6. Hence this will be loss of funds to the `User A`.
7. Hence it is recommended to either allow only the `mintPrice` to be supplied as the `msg.value` (mintPrice == msg.value) and revert the transaction if the `msg.value` is greater than the `mintPrice`. Hence the refund mechanism can be omitted in this scenario.
8. If the `excess msg.value` is allowed then it is recommended to inform the `users` of this possibility of thier transaction being front-runned and supplied excessive eth being consumed.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L195-L199
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L215-L220

## 6. The `amountMinted` variable declared in the `TraitForgeNft.mintWithBudget` function is redundant and can be omitted

The `TraitForgeNft.mintWithBudget` function declares the `amountMinted` local variable to keep track of the number of minted tokens within the function execution. This `amountMinted` variable is incremented inside the `while loop` for each token minted. But the issue is this `amountMinted` variable is not used anywhere within the function scope and hence it is a redundant implementation which can be omitted to make the code more optimized.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L212
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L217

## 7. The `TraitForgeNft.isForger` function logic implementation can be optimized

The `TraitForgeNft.isForger` function is used to compute whether the passed in `tokenId` is a `forger`. The `isForger` declares two local variables in its function scope for the logic implementation. But these two variables are not required for the logic implementation and the function logic can be optimized as shown below:

```solidity
  function isForger(uint256 tokenId) public view returns (bool) {
    return tokenEntropy[tokenId] % 3 == 0;
  }
```

The above optimized function implementation results in the same logic execution as the `isForger` implementation in the code.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L274-L278

## 8. Recommended to use the `_safeMint` function inplace of the `_mint` function in the `TraitForgeNft._mintInternal` function

The `TraitForgeNft._mintInternal` function uses the `_mint` internal function of the `ERC721.sol` contract to mint the new NFTs. But if the `to address` to whom the NFT is minted is a contract and if it does not have the funcionality to handle the `NFTs` then the minted `TraitForgeNfts` will be locked in that `to address contract` permenantly. 

Hence it is recommended to use the `_safeMint` internal function of the `ERC721.sol` contract instead in the `TraitForgeNft._mintInternal` function. The `_safeMint` will revert the transaction if the `to address` is a contract and can not handle the `NFTs` minted to it. Hence the `minter` contract will not be at a loss as a result.

Same issue is found in the `TraitForgeNft._mintNewEntity` function where the `_mint` function is used and it should be replaced with the `_safeMint` function.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L287
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L323

## 9. Redundant conditional checkn in the `TraitForgeNft._mintNewEntity` function can be optimized

The `TraitForgeNft._mintNewEntity` function is used to mint a new `NFT` via `Forging`. In the `_mintNewEntity` function the following condition is given.

```solidity
    if ( //@audit-issue - generationMintCounts[gen] > maxTokensPerGen condition can never occur here
      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration
    ) { //@audit - Why is this logic in place?
      _incrementGeneration();
    }
```

The issue here is that in the `generationMintCounts[gen] >= maxTokensPerGen` conditional check the `generationMintCounts[gen] > maxTokensPerGen` condition can never occur. This is because of the previous conditional check in the `_mintNewEntity` function at the beginning which is given below:

```solidity
    require(
      generationMintCounts[gen] < maxTokensPerGen,
      'Exceeds maxTokensPerGen'
    );
```

Since the `generationMintCounts[gen]` is incremented by 1, after the abvoe check the maximum value the `generationMintCounts[gen]` will get is `maxTokensPerGen`. Hence the `generationMintCounts[gen] >= maxTokensPerGen` check in the `_mintInternal` function should be optimized as `generationMintCounts[gen] == maxTokensPerGen`. 

The modified code snippet is as follows:

```solidity
    if ( 
      generationMintCounts[gen] == maxTokensPerGen && gen == currentGeneration
    ) {
      _incrementGeneration();
    }
```

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L332
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L316-L319

## 10. `whenNotPaused` should be added to the `TraitForgeNft._beforeTokenTransfer` function instead of checking the `paused state of the contract` at the end of the function execution

The `TraitForgeNft._beforeTokenTransfer` function is called during the `minting, burning or transfer of NFT tokens`. At the end of the `_beforeTokenTransfer` function there is a logic to revert the transaction if the contract is `paused`. But this is not efficient and optimal since the contract is checked for its `paused state` after all the logic in the `_beforeTokenTransfer` function is executed. 

Hence it is recommended `add the whenNotPaused modifier` in the `TraitForgeNft._beforeTokenTransfer` function. The modified function is shown below:

```solidity
  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 firstTokenId,
    uint256 batchSize
  ) internal virtual override whenNotPaused {
```

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L394

## 11. The `endIndex` value calculation in the `EntropyGenerator.writeEntropyBatch2` function can be optimized

In the `EntropyGenerator.writeEntropyBatch2` function the `endIndex` is calculated as follows:

```solidity
    uint256 endIndex = lastInitializedIndex + batchSize1;
```

The above `endIndex` calculation should be optimized as follows:

```solidity
    uint256 endIndex = batchSize2;
```

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L70

## 12. The `require` statement logic in the `EntropyGenerator.getNextEntropy` function is errorneous

The `EntropyGenerator.getNextEntropy` function is used to calculate the next entorpy value from the `entropySlots` array. There is `require` conditional check in the `getNextEntropy` function as shown below:

```solidity
    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
```

Here the `maxSlotIndex == 770` and the `<=` operator allows the `currentSlotIndex == maxSlotIndex` as well. This means that `currentSlotIndex == 770` is an allowed state. But there is no slot for the `770th` slot in the `entropySlots` array since the last probable index of this array = 769. If the `770` is passed in as the slot index to the `entropySlots` array the transaction will revert due to `array out of bounds` error.

Hence it is recommended to change the `require` statement check to `<` operator from the `<=` operator.
The modified `require` statement is shown below:

```solidity
    require(currentSlotIndex < maxSlotIndex, 'Max slot index reached.');
```

Similar issue is found in the `require` statement of the `EntropyGenerator.getEntropy` function where the `slotIndex == 770` is allowed which will result in `array out of bounds` error. Hence this should be corrected as explained above.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L102-L103
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L180

## 13. Multiple calls to the `nftContract.ownerOf(forgerTokenId)` function can be omitted thus making the `EntityForging.forgeWithListed` function more optimized

In the `EntityForging.forgeWithListed` function there are two calls made to the `nftContract.ownerOf(forgerTokenId)`. But the value of the `owner` can be cached after the first call and used this cached `owner` value in the second call thus making the code more optimized.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L116
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L148

## 14. The related state in the `EntityForging._cancelListingForForging` function is not fully cleared (deleted)

The `EntityForging._cancelListingForForging` function is used to cancel an already listed forgeTokenId from being forged. In the `_cancelListingForForging` function execution the entry, relating to the `forgerTokenId` of whose forge listing is being cancelled, is deleted as shown below:

```solidity
    delete listings[listedTokenIds[tokenId]]
```

But the issue is entry in the listedTokenIds mapping for the given `tokenId` (`listedTokenIds[tokenId]`) is not deleted. 

Hence it is rerecommended to delete the entry in the `listedTokenIds` mapping for the given `tokenId` as well to optimize the `_cancelListingForForging` function. The modified `EntityForging._cancelListingForForging` function is shown below:

```solidity
  function _cancelListingForForging(uint256 tokenId) internal {
    delete listings[listedTokenIds[tokenId]];
    delete listedTokenIds[tokenId];

    emit CancelledListingForForging(tokenId); // Emitting with 0 fee to denote cancellation
  } 
```

Similar issue is found in the `EntityTrading.cancelListing` function where the `listedTokenIds[tokenId]` state is not deleted after the `listing is cancelled`. Hence it is recommended to delete the `listedTokenIds[tokenId]` state as well in the `EntityTrading.cancelListing` function.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L193-L197
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L106
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L83

## 15. Ownable contract should be replaced with OpenZeppelin's `Ownable2Step` contract

The `EntityTrading` contract is inheriting from the `Ownable` contract. There are `onlyOwner` modifier controlled functions in the `EntityTrading` contract. Hence as a best it is recommended to add the following changes to the `EntityTrading` contract.

1. Recommended to overwrite the `renonceOwnership` function of the `Ownable` contract to revert when called. This will ensure the `owner` is not revoked maliciously or mistakenly. This will ensure the critical functions which are access controlled by the `owner` are not affected due to owner being revoked.

2. Recommended to replace the `Ownable` contract with the `Ownable2Step` contract. This will ensure ownership transfer will happen in 2 steps which is more secure.

Recommended to introduce the `Ownable2Step` contract (by replacing the `Ownable` contract) in the other contracts as well which inherit the `Ownable` contract.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L11

## 16. `TraitForgeNft.nukeFundAddress` state variable is not declared as a `payable address` variable.

The `nukeFundAddress` state variable in the `TraitForgeNft` contract is not declared as a `payable address` variable. In the `TraitForgeNft.setNukeFundContract` function the input parameter `_nukeFundAddress` is a `payable address` which is assigned to the `nukeFundAddress` state variable. Hence it is recommended to declare the `nukeFundAddress` state variable as a `payable address` variable.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L30
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L67-L69

## 17. No `address(0)` check is performed when the `nukeFundAddress` state variable address is set

The `nukeFundAddress` state variable is set in the `TraitForgeNft.setNukeFundContract` function. The `nukeFundAddress` is used in the `TraitForgeNft._distributeFunds` function to transfer the `funds` using the `low-level call function`. If the `low-level call` function is called on the `address(0)` the return value will always be `true` and this will be loss of funds.

Hence it is recommended to check for the `address(0)` in the `TraitForgeNft.setNukeFundContract` function to ensure that `nukeFundAddress` is not set to the `address(0)`.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L67-L69
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L361

## 18. No event is emitted after critical state variable modifications

The `entityForgingContract` state variable in the `TraitForgeNft` contract is set in the `TraitForgeNft.setEntityForgingContract` function. But there is no `event emitted` after this state modification. Hence it is recommended to emit an `event` after critical state variable modifications.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L79