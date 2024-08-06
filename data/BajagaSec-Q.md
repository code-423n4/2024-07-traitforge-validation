## Unused variable `amountMinted` in `mintWithBudget()` increase gas coast.

## Description
In the `TraitForgeNft.sol` contract, the `mintWithBudget()` function contains the variable `amountMinted`, which is incremented for every entity minted. However, this variable is unused and only increases the gas cost of the function.

```solidity
function mintWithBudget(
    bytes32[] calldata proof 
  )  public payable  whenNotPaused nonReentrant  onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  { 
    uint256 mintPrice = calculateMintPrice(); 
    -> uint256 amountMinted = 0; 
    uint256 budgetLeft = msg.value; 

    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
      _mintInternal(msg.sender, mintPrice);
    -> amountMinted++; 
      budgetLeft -= mintPrice;
      mintPrice = calculateMintPrice(); 
    } 
    if (budgetLeft > 0) { 
      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }(''); 
      require(refundSuccess, 'Refund failed.'); 
    } 
  }
  ```

## Tools Used
Manual Review

## Recommended Mitigation Steps
Remove the `amountMinted` variable from the `mintWithBudget()` function to reduce unnecessary gas costs:

```solidity
function mintWithBudget(
    bytes32[] calldata proof 
  )  public payable  whenNotPaused nonReentrant  onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  { 
    uint256 mintPrice = calculateMintPrice(); 
    - uint256 amountMinted = 0; 
    uint256 budgetLeft = msg.value; 

    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
      _mintInternal(msg.sender, mintPrice);
    - amountMinted++; 
      budgetLeft -= mintPrice;
      mintPrice = calculateMintPrice(); 
    } 
    if (budgetLeft > 0) { 
      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }(''); 
      require(refundSuccess, 'Refund failed.'); 
    } 
  }
  ```


 ## Irrelevant Check for Next Generation in  _mintNewEntity()

  ## Description
In the `TraitForgeNft.sol` contract, the `forge()` function calls the `_mintNewEntity()` function to mint a new entity in the next generation after successful forging. According to the project documentation, if the `nextGenMintCoun`t is full, entities cannot be forged. The `_mintNewEntity()` function includes a require statement that ensures this:

 ```solidity
 require( 
      generationMintCounts[gen] < maxTokensPerGen, 
      'Exceeds maxTokensPerGen' 
    );
```    


Additionally, there is an `if` statement in the same function that checks whether to increase the generation count if the current generation is full. This check is redundant because the `require` statement already excludes this possibility.

## Recommended Mitigation Steps

Remove the redundant check for increase generation in `_mintNewEntity()` function:

```solidity
function _mintNewEntity( address newOwner,uint256 entropy, uint256 gen ) private returns (uint256) {
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

 -   if ( generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration) { 
 -     _incrementGeneration(); 
 -   } 

    if (!airdropContract.airdropStarted()) {
      airdropContract.addUserAmount(newOwner, entropy); 
    } 

    emit NewEntityMinted(newOwner, newTokenId, gen, entropy); 
    return newTokenId; 
  } 

```