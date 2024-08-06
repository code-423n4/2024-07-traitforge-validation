## QA REPORT

|      | Issue                                                                                           |
| ---- | :---------------------------------------------------------------------------------------------- |
| [01] | Changing Centralized Parameters Half-way Could Cause Issues                                     |
| [02] | Whitelist Users Could Participate More Than Once                                                |
| [03] | The usage of `amountMinted` is redundant                                                        |
| [04] | The `>=` check for `generationMintCounts` could be simplified                                   |
| [05] | Self-transfer of `TraitForgeNft` Could Clear Listing Info                                       |
| [06] | Inconsistent Usage of `pause()` and `whenNotPaused` in `TraitForgeNft`                          |
| [07] | Uninitialized/Unbounded Issue in `fetchListings`                                                |
| [08] | Seller Could Deliberately Refuse A Purchase/Forge                                               |
| [09] | Relationship between `maxAllowedClaimDivisor` and `nukeFactorMaxParam` is not strictly enforced |
| [10] | The `merger`Could Send the `mergeNFT` To A New Address to Bypass the Check                      |
| [11] | Secondary Market Buyer May Suffer A Loss If the Forging Happens Before the Transfer             |
| [12] | Ambiguous Mod Due to  `2 ** 256 < 10 ** 78`                                                     |
| [13] | `deriveTokenParameters` does not align with the doc                                             |

## [01] Changing Centralized Parameters Half-way Could Cause Issues

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L22-L25
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L37-L48
### Description
In the project, several parameters such as `maxTokensPerGen`, `startPrice`, `priceIncrement`, and `priceIncrementByGen` are designed to be constants:

```solidity
  // Constants for token generation and pricing
  uint256 public maxTokensPerGen = 10000; // constants should be constants
  uint256 public startPrice = 0.005 ether;
  uint256 public priceIncrement = 0.0000245 ether;
  uint256 public priceIncrementByGen = 0.000005 ether;
```

However, these parameters, along with others like `maxGeneration` and `rootHash`, can be modified by the contract owner:

```solidity
  function setStartPrice(uint256 _startPrice) external onlyOwner {
    startPrice = _startPrice;
  }

  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {
    priceIncrement = _priceIncrement;
  }

  function setPriceIncrementByGen(
    uint256 _priceIncrementByGen
  ) external onlyOwner {
    priceIncrementByGen = _priceIncrementByGen;
  }

  function setMaxGeneration(uint maxGeneration_) external onlyOwner {
    require(
      maxGeneration_ >= currentGeneration,
      "can't below than current generation"
    );
    maxGeneration = maxGeneration_;
  }

  function setRootHash(bytes32 rootHash_) external onlyOwner {
    // @note: Centralized Risk. Should not change.
    rootHash = rootHash_;
  }

  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {
    whitelistEndTime = endTime_;
  }
```

Allowing these parameters to be changed midway through the contract’s owner's operation can lead to several unexpected consequences:

• **Price Changes**: If `startPrice`, `priceIncrement`, or `priceIncrementByGen` are suddenly altered, users could unintentionally spend more funds than anticipated.
• **Generation Limits**: Modifying `maxGeneration` can unexpectedly change the global total supply of tokens.
• **Whitelist Integrity**: Changing the `rootHash` halfway can allow more whitelisted accounts to join and mint, leading to unexpected outcomes.
• **Whitelist Period Extension: Changing the `whitelistEndTime` halfway can extend the whitelisted period.

To note, this is also the same in other contracts like `EntityForging`.
• **Fee and Tax**: If `taxCut`, `minimumListFee`are suddenly altered, users could unintentionally pay more taxes/fees than anticipated.

To note, this is also the same in other contracts like `NukeFund`.
• **Quick Exit**: If `ageMultiplier`, `minimumDaysHeld`are suddenly altered, users could `nuke` their NFT in a shorter period and get more shares.

### Recommendation
To mitigate, re-assess whether these parameters need to be modifiable. If they are intended to be constants, enforce immutability to avoid potential issues caused by centralized operations.

## [02] Whitelist Users Could Participate More Than Once

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L51-L59

### Description
The `onlyWhitelisted` modifier ensures that only whitelisted users can perform certain operations within the whitelist period:

```solidity
  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
    if (block.timestamp <= whitelistEndTime) { // @note could disable the whitelist only at anytime by the owner
      require(
@=>      MerkleProof.verify(proof, rootHash, leaf), // @note: whitelist can continously participate
        'Not whitelisted user'
      );
    }
    _;
  }
```

However, the modifier does not record the `leaf` input to prevent future replays. Consequently, whitelisted users can participate in minting more than once by providing the same proof multiple times.

### Recommendation
To mitigate this issue, revise the design to check if each whitelisted user can  participate more than once once during the whitelist period. 

If it is to be restricted, this can be achieved by recording the participation of each user and preventing multiple entries.

## [03] The usage of `amountMinted` is redundant

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L212-L213
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L217-L218
### Description
In the `mintWithBudget` function, the local variable `amountMinted` is used to increment with each mint but serves no further purpose within the function. This makes the use of `amountMinted` redundant.
```solidity
  function mintWithBudget(
    bytes32[] calldata proof
  )
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  {
	    uint256 mintPrice = calculateMintPrice();
@=> 	uint256 amountMinted = 0;
	    uint256 budgetLeft = msg.value;
	
	    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) { // @audit _tokenIds can only work with 1 generation
	      _mintInternal(msg.sender, mintPrice);
@=> 	  amountMinted++;
	      budgetLeft -= mintPrice;
	      mintPrice = calculateMintPrice();
	    }
	    if (budgetLeft > 0) {
	      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
	      require(refundSuccess, 'Refund failed.');
	    }
  }
```
The variable `amountMinted` is incremented within the while loop but is not used after the loop. Hence, it serves no purpose in the function and can be removed.
### Recommendation
Remove the `amountMinted` variable and its associated increment statement to simplify the code.

## [04] The `>=` check for `generationMintCounts` could be simplified

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L332
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L281
### Description
In the `TraitForgeNft` contract, the check `generationMintCounts[currentGeneration] >= maxTokensPerGen` can be simplified to `generationMintCounts[currentGeneration] == maxTokensPerGen`. 

Since `generationMintCounts[currentGeneration]` will not exceed `maxTokensPerGen` under normal circumstances (if `maxTokensPerGen` is not altered during the contract’s lifecycle), the check can be simplified to:

```solidity
    if (generationMintCounts[currentGeneration] >= maxTokensPerGen) {
      _incrementGeneration();
    }
```

### Recommendation
Simplify the check to `==` if it is guaranteed that `maxTokensPerGen` will not be changed during the contract’s lifecycle.

## [05] Self-transfer of `TraitForgeNft` Could Clear Listing Info

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L378-L394
### Description
In the `TraitForgeNft` contract, the `_beforeTokenTransfer` function is overridden to include additional constraints, such as canceling the listing information of a token upon transfer. However, the current implementation does not account for self-transfers (where the from and to addresses are the same). This could lead to accidental clearing of listing information.

```solidity
    /// @dev don't update the transferred timestamp if from and to address are same
    if (from != to) {
      lastTokenTransferredTimestamp[firstTokenId] = block.timestamp;
    }

    // @note: self-transfer could clear listing info.
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
```

The condition `from != to` is considered for updating the transfer timestamp `lastTokenTransferredTimestamp`, but not for canceling the listing. This oversight allows a self-transfer (where from == to) to inadvertently clear the listing information.

### Recommendation

To prevent accidental clearing of listing information during self-transfers, modify the `_beforeTokenTransfer` function to only cancel the listing if `from!=to`.

## [06] Inconsistent Usage of `pause()` and `whenNotPaused` in `TraitForgeNft`

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L207-L208
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L396-L397
### Description
The `TraitForgeNft` contract inherits from `Pausable` and supports pausing functionalities. However, the contract uses the pausing mechanism inconsistently across different functions.

In the `TraitForgeNft` contract, the `mintToken` function uses the `whenNotPaused` modifier to ensure it can only be executed when the contract is not paused.
On the other hand, the `_beforeTokenTransfer` function directly checks the `paused()` state without using the `whenNotPaused` modifier.

```solidity
  function mintToken(
    bytes32[] calldata proof
  )
    public
    payable
@=> whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  {...}

  function _beforeTokenTransfer(
    address from,
    address to,
    uint256 firstTokenId,
    uint256 batchSize
  ) internal virtual override {
	    super._beforeTokenTransfer(from, to, firstTokenId, batchSize);
		...
@=>     require(!paused(), 'ERC721Pausable: token transfer while paused'); // @note: why not use whenNotPaused
  }
```

The inconsistent usage is not a good practice and could cause confusion.
### Recommendation

To ensure consistency and improve readability, use the `whenNotPaused` modifier in the `_beforeTokenTransfer` function instead of directly checking the `paused()` state. This will align the pausing logic with other functions in the contract.

## [07] Uninitialized/Unbounded Issue in `fetchListings`

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L49-L54
### Description
The function fetchListings initializes an array of `listingCount + 1` but only fills elements from index `1` to `listingCount`, leaving the first element uninitialized. This off-by-one issue can cause confusion and may lead to unexpected behavior when the function is used.

```solidity
  function fetchListings() external view returns (Listing[] memory _listings) {
    _listings = new Listing[](listingCount + 1);
    for (uint256 i = 1; i <= listingCount; ++i) {
      _listings[i] = listings[i]; //@audit incorrect setup
    }
  }
```


Additionally, if listings are deleted via the `cancelListingForForging` function, the corresponding elements in the listings array become uninitialized or hold no value, further complicating the data returned by `fetchListings`.

**Also, since the listing is unbounded(grows over time) and is looped, when there are so many listings, this could cause performance issues (DOS) or OOG(out-of-gas) issues when all previous listings are retrieved. **
### Recommendation

- Revise the design to ensure that the returned array only contains initialized elements. One approach is to skip index `0` or filter out uninitialized elements before returning the array.
- Revise the design to keep track of the active listing to avoid oog/dos issues, or use pagination to improve performance.

## [08] Seller Could Deliberately Refuse A Purchase/Forge

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L77
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L159-L160
### Description
The `seller/forgeOwner` could deliberately refuse a purchase/forge in the callback/receive function by simply reverting. This can result in a denial of service (DoS) attack, where the `seller/forgeOwner` prevents successful transactions, causing frustration and poor user experience.

```solidity
...
    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');
...
    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(
      ''
    );
```

If the `seller` or `forgeOwner` reverts the transaction, it will cause a failure in the purchase/forge process, resulting in a denial of service for other users. Since the listing can only be canceled by the `seller/forgeOwner`, this can lead to a bad user experience.

### Recommendation

To mitigate this issue, implement a method to allow the contract owner to cancel the listing. This will provide a mechanism to handle cases where the `seller/forgeOwner` deliberately refuses transactions.

## [09] Relationship between `maxAllowedClaimDivisor` and `nukeFactorMaxParam` is not strictly enforced

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L168-L173
### Description
The nuke function in the `NukeFund` contract does not enforce the relationship between `maxAllowedClaimDivisor` and `nukeFactorMaxParam`. This relationship is crucial for the proper calculation of the `claimAmount`. If these variables are not set correctly, it can lead to incorrect or unexpected behavior in the `nuke` process, potentially resulting in incorrect distribution of funds.

In the nuke function, the `claimAmount` is calculated based on whether `finalNukeFactor` exceeds `nukeFactorMaxParam`:

```solidity
    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; // Calculate the potential claim amount based on the finalNukeFactor
    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size

    // Directly assign the value to claimAmount based on the condition, removing the redeclaration
    uint256 claimAmount = finalNukeFactor > nukeFactorMaxParam
      ? maxAllowedClaimAmount
      : potentialClaimAmount;

    fund -= claimAmount; // Deduct the claim amount from the fund
```

The implicit relationship is:
```solidity
fund / maxAllowedClaimDivisor = fund * nukeFactorMaxParam / MAX_DENOMINATOR
=>
maxAllowedClaimDivisor * nukeFactorMaxParam = MAX_DENOMINATOR
```

However, both `maxAllowedClaimDivisor` and `nukeFactorMaxParam` are set independently without enforcing this constraint:

```solidity
  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {
    maxAllowedClaimDivisor = value;
  }

  function setNukeFactorMaxParam(uint256 value) external onlyOwner {
    nukeFactorMaxParam = value;
  }
```

### Recommendation

Ensure that the relationship between `maxAllowedClaimDivisor` and `nukeFactorMaxParam` is maintained when setting these values. This can be done by modifying the setter functions to enforce the constraint.

## [11] Secondary Market Buyer May Suffer A Loss If the Forging Happens Before the Transfer

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L112-L119
### Description

The parameter `forgePotential` is crucial as it determines the number of times an NFT can forge within a year.

```solidity
    require(
      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
      'Entity has reached its forging limit'
    );
```

In the secondary market, NFTs with a higher `forgePotential` or lower `forgeCounts` are valued more. However, there is a potential issue that can cause losses for buyers in the secondary market. Consider the following scenario:

1. User `Alice` lists an NFT for forging, which can still forge one more time.
2. User `Alice` lists the NFT in the secondary market.
3. User `Bob` buys the NFT, willing to pay a higher price since the NFT can still forge.
4. Just before his purchase, the NFT is forged. Consequently, `Bob` loses money as the NFT’s value decreases due to it reaching its forging limit.

### Recommendation
- **Cooldown Period for Transfer**: Implement a cooldown period after a successful forge, during which the NFT cannot be transferred. This ensures that the buyer can verify the forging status and potential before completing the purchase.

## [12] Ambiguous Mod Due to  `2 ** 256 < 10 ** 78`

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L73-L75
### Description

The `mod` operation in the code below is unnecessary and adds unnecessary complexity. This is because the modulus value `10 ** 78` is larger than the maximum value of a uint256 (`2 ** 256`). As a result, the modulus operation has no effect, leading to ambiguity and potential misunderstanding of the code’s intent.

```solidity
        uint256 pseudoRandomValue = uint256(
          keccak256(abi.encodePacked(block.number, i))
        ) % uint256(10) ** 78;
```

### Recommendation
- Remove the unnecessary modulus operation to simplify the code and avoid confusion.

## [13] `deriveTokenParameters` does not align with the doc

### Link
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L152-L153
### Description

The function `deriveTokenParameters` contains calculations that do not align with the project’s documentation. This discrepancy can lead to misunderstandings and potential bugs in the system, as the calculated parameters will not match the expected values described in the documentation.**


The documentation specifies the following calculations:
1. **Entropy / 40 = initialNukeFactor**
2. **Entropy[4] = colour1 && forgePotential**

However, the deriveTokenParameters function calculates:
1. nukeFactor = entropy / 4000000
2. forgePotential = getFirstDigit(entropy)

```solidity
  // function to derive various parameters baed on entrtopy values, demonstrating potential cases
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
@=> nukeFactor = entropy / 4000000; // inconsistent with design
@=> forgePotential = getFirstDigit(entropy);
    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
  }
```

### Recommendation
Since the `deriveTokenParameters` function is not being used anymore, it is recommended to remove it from the codebase to avoid confusion and ensure the code remains clean and maintainable.