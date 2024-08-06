| Finding number | Title |
| ----------- | ----------- |
| QA-1 | The NFT with tokenId 0 cannot be minted |
| QA-2 | `_safeMint()` should be used instead of `_mint()` |
| QA-3 | `mintWithBudget()` has an inadequate check that will be invalid after generation 1 |
| QA-4 | `TraitForgeNft::isForger()` returns `true` even if the `tokenId` has not been minted yet |
| QA-5 | `getNextEntropy()` misses `whenNotPaused` modifier |
| QA-6 | `EntropyGenerator::deriveTokenParameters()` wrongly extracts the forge potential |
| QA-7 | `EntropyGenerator::deriveTokenParameters()` wrongly extracts the nuke factor |
| QA-8 | The owner of the forging NFT can revert the forging any time |
| QA-9 | Excess eth value is stuck when using `EntityForging::forgeWithListed()` |
| QA-10 | `safeTransferFrom` should be used instead of `transferFrom` |

###### [QA-1] The NFT with tokenId 0 cannot be minted

**Description:**

When minting a new NFT, first the `_tokenIds` state variable is increased and then we call `_mint`. This means that the first minted NFT will be with `tokenId = 1` instead of 0.

```solidity
  function _mintInternal(address to, uint256 mintPrice) internal {
    //...

    _tokenIds++;
    uint256 newItemId = _tokenIds;
    _mint(to, newItemId);
    //...
}
```

---

###### [QA-2] `_safeMint()` should be used instead of `_mint()`

**Description:**

`TraitForgeNft::_mintInternal()` uses `_mint()` instead of `_safeMint()` which doesn't ensure that the receiver of the NFT is able to accept it. Openzeppelin recommends using `_safeMint()` just to be sure.

```solidity
  function _mintInternal(address to, uint256 mintPrice) internal {
    //...

    _tokenIds++;
    uint256 newItemId = _tokenIds;
    _mint(to, newItemId);
    //...
}
```

---

###### [QA-3] `mintWithBudget()` has an inadequate check that will be invalid after generation 1

**Description:**

`TraitForgeNft::mintWithBudget()` allows minting until the maximum number of tokens per generation is minted (or the funds are spent), meaning that this function can't be used to mint NFTs of 2 generations.

`while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen)`

However this check is wrong because `tokenIds` don't restart after the generation is incremented, meaning this check will be ineffective after generation 1 because  `_tokenIds` will be way higher than `maxTokensPerGen = 10_000`.

---

###### [QA-4] `TraitForgeNft::isForger()` returns `true` even if the `tokenId` has not been minted yet

**Description:**

`TraitForgeNft::isForger()` doesn't check if the NFT has been minted before returning an answer. If this is used it could mislead people into believing that the NFT exists.

```solidity
  function isForger(uint256 tokenId) public view returns (bool) {
    uint256 entropy = tokenEntropy[tokenId];
    uint256 roleIndicator = entropy % 3;
    return roleIndicator == 0;
  }
```

---

###### [QA-5] `getNextEntropy()` misses `whenNotPaused` modifier

**Description:**

`EntropyGenerator` is a pausable contract and `TraitForgeNft` uses its `getNextEntropy()` to determine the NFT's entropy. However if the `EntropyGenerator` is paused for whatever reason, the `TraitForgeNft` contract will continue to interact with the paused contract `EntropyGenerator`. This is possible because `getNextEntropy()` misses `whenNotPaused` modifier.

```solidity
  function getNextEntropy() public onlyAllowedCaller returns (uint256) {
    //...
  }
```

---

###### [QA-6] `EntropyGenerator::deriveTokenParameters()` wrongly extracts the forge potential

**Description:**

`EntropyGenerator::deriveTokenParameters()` uses the first digit of the entropy to determine the forge potential which is wrong. The NFT's entropy is a 6-digit number and the forge potential is meant to be the 5th digit (can be seen in `EntityForging`) instead of the first.

```solidity
  function deriveTokenParameters(
    uint256 slotIndex,
    uint256 numberIndex
  )
    public
    view
    returns (
      uint256 nukeFactor,
      uint256 forgePotential,
      uint256 performanceFactor,
      bool isForger
    )
  {
    uint256 entropy = getEntropy(slotIndex, numberIndex);

    // example calcualtions using entropyto derive game-related parameters
    nukeFactor = entropy / 4000000;
@->    forgePotential = getFirstDigit(entropy);
    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
  }
```

---

###### [QA-7] `EntropyGenerator::deriveTokenParameters()` wrongly extracts the nuke factor

**Description:**

`EntropyGenerator::deriveTokenParameters()` divides by 4000000 to get the `nukeFactor` from the entropy, however the `nukeFactor` should be a 5-digit number and should be extracted by dividing the entropy by 40. This makes the function return wrong results.

```solidity
  function deriveTokenParameters(
    uint256 slotIndex,
    uint256 numberIndex
  )
    public
    view
    returns (
      uint256 nukeFactor,
      uint256 forgePotential,
      uint256 performanceFactor,
      bool isForger
    )
  {
    uint256 entropy = getEntropy(slotIndex, numberIndex);

    // example calcualtions using entropyto derive game-related parameters
@->    nukeFactor = entropy / 4000000;
    forgePotential = getFirstDigit(entropy);
    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
  }
```

---

###### [QA-8] The owner of the forging NFT can revert the forging any time

**Description:**

When a user with a merger NFT matches a forger NFT, he has to pay a fee that is later transferred to the forger NFT's owner via a low-level call. Thus the owner of the forger NFT can revert the transaction anytime he wishes and he can decide which merger to match with. This can block a user from forgin successfully and wastes his time and funds(gas spent on each transaction).

```solidity
    function forgeWithListed(
    uint256 forgerTokenId,
    uint256 mergerTokenId
  ) external payable whenNotPaused nonReentrant returns (uint256) {
    //... 

    (bool success, ) = nukeFundAddress.call{ value: devFee }('');
    require(success, 'Failed to send to NukeFund');
@->    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');
    require(success_forge, 'Failed to send to Forge Owner');

    //...
}
```

---

###### [QA-9] Excess eth value is stuck when using `EntityForging::forgeWithListed()`

**Description:**

`EntityForging::forgeWithListed()` is a payable function and the caller of this function should pay fees for a successful forging operation. This function makes sure that the send value is >= than the listing fee:

`require(msg.value >= forgingFee, 'Insufficient fee for forging');`

But doesn't send the excess amount back to `msg.sender`. This means that there can be stuck eth in the `EntityForging` contract because there is no way to get it back.

---

###### [QA-10] `safeTransferFrom` should be used instead of `transferFrom`

**Description:**

When transferring NFTs in `EnitityTrading` `safeTransferFrom` should be used instead of `transferFrom`. That is because if the receiver of the NFT isn't able to receive it, the NFT will be permanently lost.

In `Entitytrading::listNFTForSale()`
```solidity
    nftContract.transferFrom(msg.sender, address(this), tokenId);
```

In `Entitytrading::buyNFT()`
```solidity
    nftContract.transferFrom(address(this), msg.sender, tokenId);
```