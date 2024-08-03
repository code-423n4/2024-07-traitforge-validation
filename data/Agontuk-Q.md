## [L-01] Potential infinite loop and gas wastage in entropy generation functions

The `writeEntropyBatch1()`, `writeEntropyBatch2()`, and `writeEntropyBatch3()` functions generate pseudo-random values and check if the value is `999999`. If it is, the transaction reverts, which can lead to potential infinite loops and gas wastage if the value `999999` is generated repeatedly. This issue arises from the following code snippet:

[EntropyGenerator.sol#L53-L56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L53-L56)
```solidity
File: EntropyGenerator.sol
53:         uint256 pseudoRandomValue = uint256(
54:           keccak256(abi.encodePacked(block.number, i))
55:         ) % uint256(10) ** 78; // generate a  pseudo-random value using block number and index
56:         require(pseudoRandomValue != 999999, 'Invalid value, retry.');

```

### Recommendation: 
Modify the logic to regenerate the value if it is `999999` instead of reverting the transaction. This can be achieved using a `do-while` loop to ensure that the value `999999` is never used:

```solidity
uint256 pseudoRandomValue;
do {
  pseudoRandomValue = uint256(
    keccak256(abi.encodePacked(block.number, i))
  ) % uint256(10) ** 78;
} while (pseudoRandomValue == 999999);
```




## [L-02] Lack of pausable functionality restriction allows unintended state modifications in EntropyGenerator

The `EntropyGenerator` contract is designed to generate pseudo-random values for use in other contracts. It includes several functions that modify the state of the contract, such as `setAllowedCaller()`, `writeEntropyBatch1()`, `writeEntropyBatch2()`, `writeEntropyBatch3()`, `getNextEntropy()`, and `initializeAlphaIndices()`. These functions are critical for the contract's operation and involve updating important state variables.

The contract inherits from `Pausable`, which is intended to allow the contract owner to pause certain operations. However, the `whenNotPaused` modifier is not applied to any of the state-modifying functions. This oversight means that these functions can still be executed even when the contract is paused, defeating the purpose of the `Pausable` functionality.

For example, the `writeEntropyBatch1()` function writes entropy values to the `entropySlots` array. If this function is called while the contract is paused, it could lead to unintended state changes. Similarly, `setAllowedCaller()` updates the `allowedCaller` address, and `getNextEntropy()` retrieves and updates the next entropy value. These operations should be restricted when the contract is paused to ensure the integrity of the contract's state.

### Impact
The lack of `whenNotPaused` modifier on state-modifying functions allows these functions to be executed even when the contract is paused. This could lead to unintended state changes and undermine the purpose of the `Pausable` functionality. For instance, an attacker or a user could continue to modify the state of the contract despite it being paused, potentially leading to inconsistent or corrupted state.


### Recommendation: 
Add the `whenNotPaused` modifier to all state-modifying functions to ensure they cannot be executed when the contract is paused. This will enforce the intended behavior of the `Pausable` functionality.



## [L-03] Initialization of `whitelistEndTime` could be clearer in the constructor

The `whitelistEndTime` is initialized in the constructor using the expression `24 hours`. While this is valid in the current Solidity version, it is less clear and consistent compared to using `24 * 1 hours`, which explicitly shows the calculation.

References to variable names or similar should be formatted as `code snippets`. Functions must always be formatted with parentheses after their name, as in `functionName()`.

Then, provide a recommendation on how to fix or improve the code in question if necessary.
Larger code blocks must be formatted using TWO backticks instead of three:


[raitForgeNft/TraitForgeNft.sol#L61-L63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L61-L63)
```solidity
File: TraitForgeNft.sol
61:   constructor() ERC721('TraitForgeNft', 'TFGNFT') {
62:     whitelistEndTime = block.timestamp + 24 hours;
63:   }

```

Using `24 * 1 hours` improves readability and consistency, making it clear that the intention is to add 24 hours to the current block timestamp.

### Recommendation:
Update the initialization to use `24 * 1 hours` for better clarity.

```solidity
constructor() ERC721('TraitForgeNft', 'TFGNFT') {
    whitelistEndTime = block.timestamp + 24 * 1 hours;
}
```




## [L-04] Lack of token existence check can cause transaction failures in `forge` function

The `forge` function in the `TraitForgeNft` contract is responsible for creating a new token by combining two parent tokens. This function is called by the `entityForgingContract` and calculates the new token's generation and entropy based on the parent tokens. However, the function does not validate whether the provided `parent1Id` and `parent2Id` are valid token IDs. This omission can lead to transaction failures if invalid token IDs are passed, as subsequent operations assume these tokens exist.

The function starts by ensuring that the caller is the `entityForgingContract`. It then calculates the new generation for the token and checks that it does not exceed the `maxGeneration`. The entropy for the new token is calculated by averaging the entropies of the parent tokens. Finally, the new token is minted with the calculated entropy and generation.

The issue arises because the function does not check if `parent1Id` and `parent2Id` are valid token IDs using the `_exists` function. If invalid token IDs are passed, the `getTokenGeneration` and `getEntropiesForTokens` functions will fail, causing the transaction to revert. This could be caught earlier by checking the existence of the tokens, providing a clearer error message and preventing unnecessary computations.

### Recommendation 
Add checks to ensure that `parent1Id` and `parent2Id` are valid token IDs using the `_exists` function. This will prevent potential errors and improve the robustness of the contract. The following diff shows the recommended changes:

```diff
function forge(
    address newOwner,
    uint256 parent1Id,
    uint256 parent2Id,
    string memory
) external whenNotPaused nonReentrant returns (uint256) {
    require(
        msg.sender == address(entityForgingContract),
        'unauthorized caller'
    );
+    require(_exists(parent1Id), 'Parent1 token does not exist'); // Check if parent1Id is valid
+    require(_exists(parent2Id), 'Parent2 token does not exist'); // Check if parent2Id is valid

    uint256 newGeneration = getTokenGeneration(parent1Id) + 1;

    /// Check new generation is not over maxGeneration
    require(newGeneration <= maxGeneration, "can't be over max generation");

    // Calculate the new entity's entropy
    (uint256 forgerEntropy, uint256 mergerEntropy) = getEntropiesForTokens(
        parent1Id,
        parent2Id
    );
    uint256 newEntropy = (forgerEntropy + mergerEntropy) / 2;

    // Mint the new entity
    uint256 newTokenId = _mintNewEntity(newOwner, newEntropy, newGeneration);

    return newTokenId;
}
```




### [L-05] Lack of Address Validation in `_mintNewEntity` Function Can Lead to Token Loss

The `_mintNewEntity` function in the `TraitForgeNft` contract does not validate the `newOwner` address, which can lead to tokens being minted to the zero address (`address(0)`). This function is responsible for minting new tokens and assigning them to a specified owner. If an invalid address is passed, the token will be minted but will not be owned by any user, effectively locking the token in an unusable state.

Relevant code:
[TraitForgeNft.sol#L311-L343](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L311-L343)
```solidity
File: TraitForgeNft.sol
311:   function _mintNewEntity(
312:     address newOwner,
313:     uint256 entropy,
314:     uint256 gen
315:   ) private returns (uint256) {
316:     require(
317:       generationMintCounts[gen] < maxTokensPerGen,
318:       'Exceeds maxTokensPerGen'
319:     );
320: 
321:     _tokenIds++;
322:     uint256 newTokenId = _tokenIds;
323:     _mint(newOwner, newTokenId);
324: 
325:     tokenCreationTimestamps[newTokenId] = block.timestamp;
326:     tokenEntropy[newTokenId] = entropy;
327:     tokenGenerations[newTokenId] = gen;
328:     generationMintCounts[gen]++;
329:     initialOwners[newTokenId] = newOwner;
330: 
331:     if (
332:       generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration
333:     ) {
334:       _incrementGeneration();
335:     }
336: 
337:     if (!airdropContract.airdropStarted()) {
338:       airdropContract.addUserAmount(newOwner, entropy);
339:     }
340: 
341:     emit NewEntityMinted(newOwner, newTokenId, gen, entropy);
342:     return newTokenId;
343:   }

```

### Recommendation: 
Add a check to ensure `newOwner` is not the zero address to prevent minting tokens to an invalid address.

```diff
function _mintNewEntity(
    address newOwner,
    uint256 entropy,
    uint256 gen
) private returns (uint256) {
+    require(newOwner != address(0), 'Invalid new owner address');
    require(
        generationMintCounts[gen] < maxTokensPerGen,
        'Exceeds maxTokensPerGen'
    );

    _tokenIds++;
    uint256 newTokenId = _tokenIds;
    _mint(newOwner, newTokenId);

    tokenCreationTimestamps[newTokenId] = block.timestamp;
    tokenEntropy[newTokenId] = entropy;
    tokenGenerations[newTokenId] = gen;
    generationMintCounts[gen]++;
    initialOwners[newTokenId] = newOwner;

    if (
        generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration
    ) {
        _incrementGeneration();
    }

    if (!airdropContract.airdropStarted()) {
        airdropContract.addUserAmount(newOwner, entropy);
    }

    emit NewEntityMinted(newOwner, newTokenId, gen, entropy);
    return newTokenId;
}
```




## [L-06] Inconsistent token existence checks in public view functions

Several public view functions in the TraitForgeNft contract have inconsistent or missing checks for token existence before accessing token-specific data. This can lead to potential return of misleading data for non-existent tokens and inconsistent behavior across different functions in the contract.

Affected functions include `isForger()`, `getTokenEntropy()`, `getTokenLastTransferredTimestamp()`, `getTokenCreationTimestamp()`, `getTokenGeneration()`, and `getEntropiesForTokens()`.

For example, `isForger()` doesn't check for token existence:
[TraitForgeNft.sol#L274-L278](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L274-L278)

```solidity
File: TraitForgeNft.sol
274:   function isForger(uint256 tokenId) public view returns (bool) {
275:     uint256 entropy = tokenEntropy[tokenId];
276:     uint256 roleIndicator = entropy % 3;
277:     return roleIndicator == 0;
278:   }

```

While `getTokenEntropy()` uses `ownerOf()` to check existence:
[TraitForgeNft.sol#L234-L240](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L234-L240)
```solidity
File: TraitForgeNft.sol
234:   function getTokenEntropy(uint256 tokenId) public view returns (uint256) {
235:     require(
236:       ownerOf(tokenId) != address(0),
237:       'ERC721: query for nonexistent token'
238:     );
239:     return tokenEntropy[tokenId];
240:   }

```

### Recommendation: 
Implement consistent token existence checks using the `ownerOf()` function at the beginning of each affected function. For example:

```solidity
function isForger(uint256 tokenId) public view returns (bool) {
    require(ownerOf(tokenId) != address(0), 'ERC721: query for nonexistent token');
    uint256 entropy = tokenEntropy[tokenId];
    uint256 roleIndicator = entropy % 3;
    return roleIndicator == 0;
}
```

This will improve the contract's consistency and prevent potential issues with non-existent tokens. Note that while using `ownerOf()` for existence checks is less gas-efficient than a dedicated `_exists()` function, it's the best available option given the current contract structure.


## [L-07] Predictable entropy calculation may allow minor manipulation of token roles

The `EntropyGenerator` contract uses the `getNextEntropy()` function to retrieve entropy values from the `entropySlots` array. These values are generated using `keccak256` with the current block number and index. The `TraitForgeNft` contract then uses these entropy values to determine if a token is a forger based on the modulo operation `(entropy % 3) == 0`.

While the use of `keccak256` with the block number and index provides some randomness, it may still be predictable if an attacker can influence or predict the block number. This could allow minor manipulation of the entropy values, potentially affecting the distribution of token roles.

> Relevant code:
[EntropyGenerator.sol#L47-L61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L47-L61)
```solidity
File: EntropyGenerator.sol
47:   function writeEntropyBatch1() public {
48:     require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');
49: 
50:     uint256 endIndex = lastInitializedIndex + batchSize1; // calculate the end index for the batch
51:     unchecked {
52:       for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
53:         uint256 pseudoRandomValue = uint256(
54:           keccak256(abi.encodePacked(block.number, i))
55:         ) % uint256(10) ** 78; // generate a  pseudo-random value using block number and index
56:         require(pseudoRandomValue != 999999, 'Invalid value, retry.');
57:         entropySlots[i] = pseudoRandomValue; // store the value in the slots array
58:       }
59:     }
60:     lastInitializedIndex = endIndex;
61:   }

```

### Recommendation: 
Enhance the entropy generation process by incorporating additional sources of randomness, such as block difficulty and timestamp, to reduce predictability.

```solidity
function writeEntropyBatch1() public {
  require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');

  uint256 endIndex = lastInitializedIndex + batchSize1; // calculate the end index for the batch
  unchecked {
    for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
      uint256 pseudoRandomValue = uint256(
        keccak256(abi.encodePacked(block.number, block.timestamp, block.difficulty, i))
      ) % uint256(10) ** 78; // generate a pseudo-random value using block number, timestamp, difficulty, and index
      require(pseudoRandomValue != 999999, 'Invalid value, retry.');
      entropySlots[i] = pseudoRandomValue; // store the value in the slots array
    }
  }
  lastInitializedIndex = endIndex;
}
```


## [L-08] Forge potential calculation uses limited entropy, potentially reducing randomness

The `EntityForging` contract's `listForForging()` function calculates the forge potential of a token using a single digit from the entropy value:

[EntityForging.sol#L86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L86)
```solidity
File: EntityForging.sol
86:     uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy

```

This method only utilizes a small portion of the available entropy, potentially reducing the randomness of the forge potential. While not immediately exploitable, this approach may lead to a less diverse distribution of forge potentials than intended, slightly impacting the fairness of the forging process.

### Recommendation: 
To improve randomness, consider using more of the entropy value or implementing a more robust randomness generation method. For example:

```solidity
uint8 forgePotential = uint8(uint256(keccak256(abi.encodePacked(entropy))) % 10) + 1;
```

This change uses the full entropy value and a hash function to generate a more unpredictable forge potential, ensuring a more even distribution of values between 1 and 10.


## [L-09] Lack of generation verification for newly forged tokens in EntityForging contract

The `EntityForging` contract's `forgeWithListed()` function checks for matching generations between the forger and merger tokens but does not verify the generation of the newly created token. While the contract delegates the responsibility of updating the new token's generation to the `ITraitForgeNft` contract's `forge()` function, there is no subsequent verification to ensure this update occurs correctly.

This lack of verification could potentially lead to generation mismatches if the `ITraitForgeNft` contract does not properly handle the generation update. Such mismatches might result in inconsistencies or exploits in future forging operations.

> Relevant code:
[EntityForging.sol#L150-L155](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L150-L155)
```solidity
File: EntityForging.sol
150:     uint256 newTokenId = nftContract.forge(
151:       msg.sender,
152:       forgerTokenId,
153:       mergerTokenId,
154:       ''
155:     );

```
### Recommendation: 
To mitigate this risk, it is recommended to add a verification step after the forging process:

```solidity
uint256 newTokenId = nftContract.forge(
    msg.sender,
    forgerTokenId,
    mergerTokenId,
    ''
);

require(
    nftContract.getTokenGeneration(newTokenId) ==
        nftContract.getTokenGeneration(forgerTokenId) + 1,
    'Invalid new token generation'
);
```

This additional check ensures that the new token's generation is correctly incremented, maintaining the integrity of the forging process and preventing potential exploits related to generation mismatches.



## [L-10] Entropy leakage in EntityForged event may expose sensitive information

The `EntityForged` event in the `EntityForging` contract emits the `newEntropy` value, potentially exposing sensitive information about the entropy used in the forging process. This could theoretically be exploited to predict or manipulate future forging operations if the entropy generation mechanism is not sufficiently secure.

The event is emitted in the `forgeWithListed()` function:
[EntityForging.sol#L166-L172](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L166-L172)
```solidity
File: EntityForging.sol
166:     emit EntityForged(
167:       newTokenId,
168:       forgerTokenId,
169:       mergerTokenId,
170:       newEntropy,
171:       forgingFee
172:     );

```

While the impact is likely minimal if the entropy source is secure, it's generally good practice to avoid exposing unnecessary information.

### Recommendation: 
Remove the `newEntropy` parameter from the `EntityForged` event to prevent potential information leakage. Update the event emission as follows:

```diff
emit EntityForged(
    newTokenId,
    forgerTokenId,
    mergerTokenId,
-   newEntropy,
    forgingFee
);
```


## [L-11] Lack of upper limit on forging fee may lead to DOS

The [`EntityForging::listForForging()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L67-L100) function allows users to set any fee above `minimumListFee` when listing a token for forging. Without an upper limit, a malicious user could set an extremely high fee, making it impractical for others to pay and effectively blocking the forging process for that token.

### Recommendation: 
To mitigate this, consider implementing an upper limit on the forging fee:

```solidity
function listForForging(uint256 tokenId, uint256 fee) public whenNotPaused nonReentrant {
    // ... existing code ...
    uint256 maxListFee = 100 ether;
    require(fee >= minimumListFee && fee <= maxListFee, "Fee out of allowed range");
    // ... rest of the function
}
```

This ensures the fee remains within a reasonable range, preventing potential abuse while maintaining flexibility in fee setting.


## [L-12] Potential front-running vulnerability in NFT purchase function

The `buyNFT()` function in the EntityTrading contract is potentially vulnerable to front-running attacks. An attacker could monitor the mempool for pending `buyNFT()` transactions and submit their own transaction with a higher gas price, allowing them to purchase the NFT before the original buyer. This could result in the original buyer's transaction failing and a loss of gas fees.

While this is a known limitation of the Ethereum transaction model rather than a contract-specific bug, it can negatively impact user experience and fairness in the NFT marketplace.

> Relevant code:
[EntityTrading.sol#L63-L92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L63-L92)

```solidity
File: EntityTrading.sol
63:   function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
64:     Listing memory listing = listings[listedTokenIds[tokenId]];
65:     require(
66:       msg.value == listing.price,
67:       'ETH sent does not match the listing price'
68:     );
69:     require(listing.seller != address(0), 'NFT is not listed for sale.');

    // ... (ETH transfer and NFT transfer logic)

83:     delete listings[listedTokenIds[tokenId]]; // remove listing
84: 
85:     emit NFTSold(
86:       tokenId,
87:       listing.seller,
88:       msg.sender,
89:       msg.value,
90:       nukeFundContribution
91:     ); // emit an event for the sale

}
```

### Recommendation: 
Consider implementing a commit-reveal scheme to mitigate front-running risks. This would involve a two-step process where buyers first commit to a purchase by submitting a hash of the purchase details, followed by a reveal phase where they disclose the actual details. While this adds complexity, it can enhance fairness in high-value or competitive NFT 
sales. Additionally, educate users about potential front-running risks and advise them on best practices for transaction timing and gas price management.


## [L-13] Lack of duplicate listing check in `EntityTrading` contract allows multiple listings of the same token

The `listNFTForSale()` function in the `EntityTrading` contract does not verify if a token is already listed before creating a new listing. This oversight allows users to list the same token multiple times, potentially leading to inconsistent contract state and user confusion. While not a critical vulnerability, this issue can impact the overall user experience and the contract's data integrity.

The current implementation in `listNFTForSale()` only checks for ownership, approval, and a valid price:
[EntityTrading.sol#L38-L60](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L38-L60)
```solidity
File: EntityTrading.sol
38:   function listNFTForSale(
39:     uint256 tokenId,
40:     uint256 price
41:   ) public whenNotPaused nonReentrant {
42:     require(price > 0, 'Price must be greater than zero');
43:     require(
44:       nftContract.ownerOf(tokenId) == msg.sender,
45:       'Sender must be the NFT owner.'
46:     );
47:     require(
48:       nftContract.getApproved(tokenId) == address(this) ||
49:         nftContract.isApprovedForAll(msg.sender, address(this)),
50:       'Contract must be approved to transfer the NFT.'
51:     );

  // ... rest of the function
}
```
### Recommendation: 
To prevent duplicate listings, it is recommended to add a check using the `listedTokenIds` mapping:

```solidity
require(listedTokenIds[tokenId] == 0, 'Token is already listed for sale');
```

This additional check ensures that each token can only have one active listing at a time, maintaining a consistent state within the contract.


## [L-14] Potential precision loss in token age calculation

The `calculateAge()` function in the `NukeFund` contract may experience precision loss due to integer division when calculating the token's age. The function divides by 365 after multiplying by large numbers (`MAX_DENOMINATOR` and `ageMultiplier`), which can lead to truncation of significant digits.

### Recommendation: 
To improve precision, consider adjusting the calculation as follows:

```solidity
function calculateAge(uint256 tokenId) public view returns (uint256) {
    // ... existing checks ...
    uint256 daysOld = (block.timestamp - nftContract.getTokenCreationTimestamp(tokenId)) / 1 days;
    uint256 performanceFactor = (nftContract.getTokenEntropy(tokenId) % 10) + 1;
    uint256 age = (daysOld * performanceFactor * MAX_DENOMINATOR * ageMultiplier) / 36500;
    return age;
}
```

This modification uses 36500 as the divisor to account for the `MAX_DENOMINATOR` scaling, reducing precision loss while maintaining the intended calculation logic.