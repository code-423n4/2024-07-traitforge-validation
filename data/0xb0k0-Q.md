# [NC-1] Solidity pragma should be specific, not wide

Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.20;`, use `pragma solidity 0.8.20;`

<details>

<summary> Instances (12) </summary>

```bash
Found in contracts/DevFund/DevFund.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L2)

```bash
Found in contracts/DevFund/IDevFund.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/IDevFund.sol#L2)

```bash
Found in contracts/EntityForging/EntityForging.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L2)

```bash
Found in contracts/EntityForging/IEntityForging.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/IEntityForging/EntityForging.sol#L2)

```bash
Found in contracts/EntityTrading/EntityTrading.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L2)

```bash
Found in contracts/EntityTrading/IEntityTrading.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/IEntityTrading.sol#L2)

```bash
Found in contracts/EntropyGenerator/EntropyGenerator.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L2)

```bash
Found in contracts/EntropyGenerator/IEntropyGenerator.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/IEntropyGenerator.sol#L2)

```bash
Found in contracts/NukeFund/NukeFund.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L2)

```bash
Found in contracts/NukeFund/INukeFund.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/INukeFund.sol#L2)

```bash
Found in contracts/TraitForgeNft/TraitForgeNft.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L2)

```bash
Found in contracts/TraitForgeNft/ITraitForgeNft.sol
2: pragma solidity ^0.8.20;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/ITraitForgeNft.sol#L2)

</details>

# [NC-2] The `nonReentrant` modifier should occur before all other modifiers

This is a best practice to protect against reentrancy in other modifiers.

<details>

<summary> Instances(12) </summary>

```bash
Found in contracts/DevFund/DevFund.sol
61: function claim() external whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L61)

```bash
Found in contracts/EntityForging/EntityForging.sol
70:  function listForForging(uint256 tokenId, uint256 fee) public whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L70)

```bash
Found in contracts/EntityForging/EntityForging.sol
105: function forgeWithListed(uint256 forgerTokenId, uint256 mergerTokenId) external payable whenNotPaused nonReentrant returns (uint256) {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L102C3-L105C68)

```bash
Found in contracts/EntityForging/EntityForging.sol
179: function cancelListingForForging(uint256 tokenId) external whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L179)

```bash
Found in contracts/EntityTrading/EntityTrading.sol
41: function listNFTForSale(uint256 tokenId, uint256 price) public whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L41)

```bash
Found in contracts/EntityTrading/EntityTrading.sol
63: function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L63)

```bash
Found in contracts/EntityTrading/EntityTrading.sol
94: function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L94)

```bash
Found in contracts/EntropyGenerator/EntropyGenerator.sol
266: function initializeAlphaIndices() public whenNotPaused onlyOwner {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L266)

```bash
Found in contracts/NukeFund/NukeFund.sol
153: function nuke(uint256 tokenId) public whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L153)

```bash
Found in contracts/TraitForgeNft/TraitForgeNft.sol
141: function burn(uint256 tokenId) external whenNotPaused nonReentrant {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L141)

```bash
Found in contracts/TraitForgeNft/TraitForgeNft.sol
158: function forge(address newOwner, uint256 parent1Id, uint256 parent2Id, string memory) external whenNotPaused nonReentrant returns (uint256) {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L158)

```bash
Found in contracts/TraitForgeNft/TraitForgeNft.sol
187: function mintToken(bytes32[] calldata proof) public payable whenNotPaused nonReentrant onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender))) {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L187)

```bash
Found in contracts/TraitForgeNft/TraitForgeNft.sol
208: function mintWithBudget(bytes32[] calldata proof) public payable whenNotPaused nonReentrant onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender))) {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L208)

</details>

# [NC-3] Modifiers invoked only once can be shoe-horned into the function

<details>

<summary> Instances(2) </summary>

```bash
Found in contracts/EntropyGenerator/EntropyGenerator.sol
25: modifier onlyAllowedCaller() {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L25C3-L25C33)

```bash
Found in contracts/TraitForgeNft/TraitForgeNft.sol
51: modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L51C3-L51C69)

</details>

# [NC-4] Internal functions called only once can be inlined

<details>

<summary> Instances(1) </summary>

```bash
Found in contracts/DevFund/DevFund.sol
83: function safeRewardTransfer(address to, uint256 amount) internal returns (uint256) {
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L83)

[NC-5] Using unsafe and outdated OZ version

The `TraitForge` protocol utilizes the [OZ contracts package](https://github.com/OpenZeppelin/openzeppelin-contracts), however the version utilized is not the latest one and is marked as having a potential issue [here](https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.9.3). Even though that the listed vulnerability is not present in the contract, it is best practice to use the latest versions of packages.

# [L-1] Listing and delisting tokens in `EntityTrading.sol` extends nuking timeout

Whenever users mint or receive a new token they need to wait 3 days before being able to nuke. However, if they list their tokens for sale and choose to delist them, their nuking timeout will be reset and they will need to wait 3 more days before being able to nuke.

```solidity
  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {
__SNIP__
    // Assuming tokenAgeInSeconds is the age of the token since it's holding the nft, check if it's over minimum days held
@>    return tokenAgeInSeconds >= minimumDaysHeld;
  }
```

```solidity
function _beforeTokenTransfer(...) internal virtual override {
__SNIP__
    /// @dev don't update the transferred timestamp if from and to address are same
    if (from != to) {
@>      lastTokenTransferredTimestamp[firstTokenId] = block.timestamp; // Updates the token held timestamp even on marketplace listing and delisting
    }
__SNIP__
  }

This can be mitigated by checking the `to` and `from` and seeing if it matches the `EntityTrading` contract address.

# [L-2] Incorrect view functions present in the contract

Both `EntityForging.sol` and `EntropyGenerator.sol` contracts contain view functions that produce incorrect results:

```solidity
  function fetchListings() external view returns (Listing[] memory _listings) {
    _listings = new Listing[](listingCount + 1);
    for (uint256 i = 1; i <= listingCount; ++i) {
      _listings[i] = listings[i];
    }
  }
```

This view function will always add an additional empty listing at the beginning of the `_listings` array and at the end.

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
@>    nukeFactor = entropy / 4000000;
    forgePotential = getFirstDigit(entropy);
    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
  }
```

As per the documentation the `forgePotential` should be calculated as `entropy / 40`.

# [L-3] The `Golden God` NFT can appear twice in one generation

The `Golden God` entity in the `TraitForge` protocol is a unique entity that has the highest entropy of `999999` and is the most sought-after of all entities. Based on the protocol documentation this entity should only appear once per generation, however there is a small chance for it to appear twice:

Imagine the following scenario:

1. `slotIndexSelectionPoint = 600`, `numberIndexSelectionPoint = 5`, and `entropySlots[700] = 123456999999123456789012345678901234567890123456789012345678901234567890`
2. `getNextEntropy()` is called.

Call 1: currentSlotIndex = 0, currentNumberIndex = 0

```solidity
getEntropy(0, 0):
  position = 0 * 6 = 0
  slotValue = entropySlots[0] (let's say it's 987654321987654321...)
  entropy = (987654321987654321... / 10^72) % 1000000 = 987654
  paddedEntropy = 987654 * (10^(6 - 6)) = 987654
Return: 987654
```

...(many calls later)...

Call 7806: currentSlotIndex = 600, currentNumberIndex = 5

```solidity
getEntropy(600, 5):
  if (600 == slotIndexSelectionPoint && 5 == numberIndexSelectionPoint):
    return 999999
Return: 999999
```

Call 9101: currentSlotIndex = 700, currentNumberIndex = 6

```solidity
getEntropy(700, 6):
  position = 6 * 6 = 36
  slotValue = 123456999999123456789012345678901234567890123456789012345678901234567890
  entropy = (123456999999123456789012345678901234567890123456789012345678901234567890 / 10^(72-36)) % 1000000
          = (123456999999123456... / 10^36) % 1000000
          = 999999
  paddedEntropy = 999999 * (10^(6 - 6)) = 999999
Return: 999999
```

From the above, it can be seen that in a very small percentage of entropies, the `Golden God` can be produced twice. The chance is around 1%. This is a tough issue to handle, but maybe an easy solution will be to have a kind of storage to keep track of how many `Golden Gods` have been produced per generation.

# [L-4] Possible front-running attack of the `Golden God`

The front-running opportunity arises because the `initializeAlphaIndices()` function uses `block.timestamp` and the previous block's hash to determine the special entropy point. A miner or a user could potentially:

1. The attacker would need to monitor the mempool for calls to `_incrementGeneration()`.
2. They would need to quickly calculate the potential `slotIndexSelectionPoint` and `numberIndexSelectionPoint` based on the current block data.
3. If their calculation shows a favorable outcome, they would submit a minting transaction with high gas fees to try to get it included before the `_incrementGeneration()` transaction.

This attack is complex and would require precise timing and calculation, but it's theoretically possible. The use of easily predictable values (`blockhash` and `block.timestamp`) for generating these points makes it vulnerable to such attacks. The most optimum thing to do is to use Chainlink VRF for proper random number generation.

# [L-5] Users can be potentially DoS-ed when bulk minting and forced to pay high gas fees

The `TraitForgeNft.sol` contract allows users to provide more ETH and mint multiple NFTs in one transaction, however, as the minting is unbounded if the user provides a lot of ETH, the transaction could revert due to `OutOfGas` issues, reverting all of the mints and forcing the user to pay the gas fees without receiving anything.

```solidity
unction mintWithBudget(
    bytes32[] calldata proof
  )
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  {
__SNIP__
@>    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) { // OutOfGas issues
      _mintInternal(msg.sender, mintPrice);
      amountMinted++;
      budgetLeft -= mintPrice;
      mintPrice = calculateMintPrice();
    }
__SNIP__
  }
```

This can be mitigated by setting a batch limit, allowing for the user to mint up to a specific number of NFTs.