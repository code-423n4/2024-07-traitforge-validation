
# Zero address check missing in `TraitForgeNft::setNukeFundContract()`

This is inconsistant with `setEntityForgingContract()`, `setEntropyGenerator()`, and `setAirdropContract()` from the same contract which all revert if the zero address is provided.

```javascript
  function setNukeFundContract(
    address payable _nukeFundAddress
  ) external onlyOwner {
    nukeFundAddress = _nukeFundAddress;
    emit NukeFundContractUpdated(_nukeFundAddress);
  }

  // Function to set the entity merging (breeding) contract address, restricted to the owner
  function setEntityForgingContract(
    address entityForgingAddress_
  ) external onlyOwner {
@>  require(entityForgingAddress_ != address(0), 'Invalid address');

    entityForgingContract = IEntityForging(entityForgingAddress_);
  }

  function setEntropyGenerator(
    address entropyGeneratorAddress_
  ) external onlyOwner {
@>  require(entropyGeneratorAddress_ != address(0), 'Invalid address');

    entropyGenerator = IEntropyGenerator(entropyGeneratorAddress_);
  }

  function setAirdropContract(address airdrop_) external onlyOwner {
@>  require(airdrop_ != address(0), 'Invalid address');

    airdropContract = IAirdrop(airdrop_);
  }
```


# In `TraitForgeNft::mintWithBudget()`, the `amountMinted` variable is initialized and updated but not used

Consider removing this variable as it is a waste of gas to updating it in memory without serving any purpose.

```javascript
  function mintWithBudget( // Allows a user to mint several NFTs until a provided mint budget is reached
    bytes32[] calldata proof
  )
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  {
    uint256 mintPrice = calculateMintPrice();
@>  uint256 amountMinted = 0;
    uint256 budgetLeft = msg.value; 

    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
      _mintInternal(msg.sender, mintPrice);
@>    amountMinted++;
      budgetLeft -= mintPrice;
      mintPrice = calculateMintPrice();
    }
    if (budgetLeft > 0) {
      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
      require(refundSuccess, 'Refund failed.');
    }
  }
```


# Owner is able to update `TraitForgeNft::maxGeneration` to the same generation which is a waste of gas

In `TraitForgeNft::setMaxGeneration()` consider requiring `maxGeneration_` to strictly exceed `currentGeneration` to avoid writing to storage for no reason.

```javascript

  function setMaxGeneration(uint maxGeneration_) external onlyOwner {
    require(
@>    maxGeneration_ >= currentGeneration,
      "can't below than current generation"
    );
    maxGeneration = maxGeneration_;
  }
```


# Remove unused input argument in `TraitForgeNft::forge()`

```javascript
  function forge(
    address newOwner,
    uint256 parent1Id,
    uint256 parent2Id,
@>  string memory 
  ) external whenNotPaused nonReentrant returns (uint256) {

    ... SKIP!...

    }
```


# It is recommended to standardize return variable declarations for consistancy

Some functions explicitly declare the return variable with a `return` function, others implicitly return variables by declaring it initially. Standardization is recommended for readability and consistancy.

Example 1:
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L227C1-L232C4
```javascript
  function calculateMintPrice() public view returns (uint256) {
    uint256 currentGenMintCount = generationMintCounts[currentGeneration];
    uint256 priceIncrease = priceIncrement * currentGenMintCount;
    uint256 price = startPrice + priceIncrease;
    return price;
  }
```

Example 2:
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L246-L252
```javascript
  function getEntropiesForTokens(
    uint256 forgerTokenId,
    uint256 mergerTokenId
  ) public view returns (uint256 forgerEntropy, uint256 mergerEntropy) {
    forgerEntropy = getTokenEntropy(forgerTokenId);
    mergerEntropy = getTokenEntropy(mergerTokenId);
  }
```


# Simplify `TraitForgeNft::isForger()` to save gas by avoiding writing to memory

```diff
  function isForger(uint256 tokenId) public view returns (bool) {
-   uint256 entropy = tokenEntropy[tokenId];
-   uint256 roleIndicator = entropy % 3;
+   return tokenEntropy[tokenId] % 3 == 0;
-    return roleIndicator == 0;
  }
```


# Inconsistancy between calculation of `forgePotential` parameter between functions

Consider the below two functions from the `EntityForging` contract, which calculate `forgePotential` as the 5th digit from the entropy. Note how these definitions differ from `EntropyGenerator::deriveTokenParameters()`.

```javascript
  function listForForging(
    uint256 tokenId,
    uint256 fee
  ) public whenNotPaused nonReentrant {
    ... SKIP!...

    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy
    
    ... SKIP!...
  }

  function forgeWithListed(
    uint256 forgerTokenId,
    uint256 mergerTokenId 
  ) external payable whenNotPaused nonReentrant returns (uint256) {
    ... SKIP!...

    uint8 mergerForgePotential = uint8((mergerEntropy / 10) % 10); // Extract the 5th digit from the entropy
    
    ... SKIP!...
  }
```

```javascript
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
    ... SKIP!...

    forgePotential = getFirstDigit(entropy); // inconsistancy between forgePotential here and in the `EntityForging::listForForging()
    
    ... SKIP!...
  }
```



# Remove redundant `require()` statement in `EntityTrading::buyNFT()` to save gas

The second `require()` statement below can never revert if the first `require()` statement does not. This is because the `listing` variable needs to exist for the first `require()` statement to pass, and the listing cannot be created with a zero address.

```javascript
  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {

    Listing memory listing = listings[listedTokenIds[tokenId]];
    require(
      msg.value == listing.price, 
      'ETH sent does not match the listing price'
    );
@>  require(listing.seller != address(0), 'NFT is not listed for sale.'); // redundant check

    ... SKIP!...
  }
```


# In the `NukeFund` contract, `nukeFactorMaxParam` should be a function of `maxAllowedClaimDivisor` but they have individual setter functions

This opens up the possibility of a mistake in setting these values resulting in unexpected nuke payouts from the `NukeFund`. For example if `setMaxAllowedClaimDivisor()` is called without calling `setNukeFactorMaxParam()` the maximum amount claimable from the `NukeFund` would behave unexpectedly.

Currently there are two separate setter functions for these state variables, it is recommended one of these is removed and is derived when the `maxAllowedClaimDivisor` is set. You may want to also consider changing the input to `setMaxAllowedClaimDivisor()` to be a uint8 or adding a require state to ensure the input contains reasonable values.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L75-L81
```diff
  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {
    maxAllowedClaimDivisor = value;
+   nukeFactorMaxParam = MAX_DENOMINATOR / maxAllowedClaimDivisor;
  }

- function setNukeFactorMaxParam(uint256 value) external onlyOwner {
-   nukeFactorMaxParam = value;
- }
```