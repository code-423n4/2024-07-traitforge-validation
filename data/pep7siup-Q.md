# Quality Assurance Report

## Table of Contents
|Issue ID|Description|
|--------|-----------|
|[QA-01](#qa-01-unbounded-loop-in-fetchlistings-causing-dos)|Unbounded loop in fetchListings causing DOS|
|[QA-02](#qa-02-wrong-require-statement-for-maxslotindex-out-of-bound)|Wrong require statement for maxSlotIndex out-of-bound|
|[QA-03](#qa-03-wrong-derivetokenparameters-reading-of-nukefactor)|Wrong deriveTokenParameters reading of nukeFactor|
|[QA-04](#qa-04-semetically-wrong-whitelistendtime)|Semantically wrong whitelistEndTime|
|[QA-05](#qa-05-mintwithbuget-will-stop-working-after-the-first-generation)|mintWithBuget will stop working after the first generation|
|[QA-06](#qa-06-amountminted-is-not-used-for-any-purpose)|amountMinted is not used for any purpose|

## [QA-01] Unbounded loop in fetchListings causing DOS
### Impact
The unbounded loop in the `fetchListings` function can lead to a Denial of Service (DoS) vulnerability. As `listingCount` increases, especially with potentially thousands of listings, the loop may exceed the block gas limit, causing failures when attempting to fetch listings. Although view functions do not consume gas when called locally (e.g., using `eth_call`), they still need to execute within the block gas limit when run on an Ethereum node. This can render any external entities relying on `fetchListings` data inoperable.
### Instances
- Found in [contracts/EntityForging/EntityForging.sol at Line 50](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L50)

```solidity
48:  function fetchListings() external view returns (Listing[] memory _listings) {
49:    _listings = new Listing[](listingCount + 1);
50:@>     for (uint256 i = 1; i <= listingCount; ++i) { 
51:      _listings[i] = listings[i];
    ...
53:  }
```
### Recommendation
Pagination or limit the number of listings fetched in a single call to ensure that the gas limit is not exceeded.

## [QA-02] Wrong require statement for maxSlotIndex out-of-bound
### Impact
The `require` statement that checks `currentSlotIndex <= maxSlotIndex` and `slotIndex <= maxSlotIndex` are incorrect. Although this issue is mitigated because `currentSlotIndex` is reset elsewhere, it still introduces a low-severity risk of out-of-bound indexing.
### Instances
- Found in [contracts/EntropyGenerator/EntropyGenerator.sol at Line 102](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L102)
> @>: currentSlotIndex == maxSlotIndex would cause out-of-bound indexing but still accepted in require check 

```solidity
101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) { 
102:@>     require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.'); 
103:    uint256 entropy = getEntropy(currentSlotIndex, currentNumberIndex);
    ...
120:  }
```
- Found in [contracts/EntropyGenerator/EntropyGenerator.sol at Line 168](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L168)
> @>: slotIndex == maxSlotIndex would cause out-of-bound indexing but still accepted in require check

```solidity
164:  function getEntropy(
    ...
167:  ) private view returns (uint256) {
168:@>     require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');
169:
    ...
185:  }
```
### Recommendation
Update the `require` statements to `require(currentSlotIndex < maxSlotIndex)` and `require(slotIndex < maxSlotIndex` accordingly.

## [QA-03] Wrong deriveTokenParameters reading of nukeFactor
### Impact
The calculation of `nukeFactor` always reads as 0 due to the entropy value being less than the divisor. This can lead to incorrect game-related parameters being derived, impacting the functionality that relies on this value.
### Instances
- Found in [contracts/EntropyGenerator/EntropyGenerator.sol at Line 152](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L152)

```solidity
136:  function deriveTokenParameters(
    ...
151:    // example calculations using entropy to derive game-related parameters
152:@>     nukeFactor = entropy / 4000000; 
153:    forgePotential = getFirstDigit(entropy);
    ...
161:  }
```

## [QA-04] Semantically wrong whitelistEndTime
### Impact
The `onlyWhitelisted` modifier allows access if `block.timestamp` is less than or equal to `whitelistEndTime`, which is semantically incorrect since the `whitelistEndTime` name semantically depicts the whitelist priority period has officially ended. This can lead to situations where non-whitelisted users can participate in a mint race, causing potential loss of funds and missed opportunities for those adhering to whitelist timings.
### Instances
- Found in [contracts/TraitForgeNft/TraitForgeNft.sol at Line 52](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L52)

```solidity
51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
52:@>     if (block.timestamp <= whitelistEndTime) { 
53:      require(
        ...
59:  }
```
### Recommendation
Change the condition to `block.timestamp < whitelistEndTime`.

## [QA-05] mintWithBuget will stop working after the first generation
### Impact
The `mintWithBudget` function's while loop condition will fail after the first generation of tokens has been minted. This breaks the core functionality of the function, preventing it from being used as intended for subsequent generations.
### Instances
- Found in [contracts/TraitForgeNft/TraitForgeNft.sol at Line 215](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L215)
> @>: while loop condition of `_tokenIds < maxTokensPerGen` would fail after the first generation has been minted. I would argue it's not intended design since mintWithBudget is designed as an alternative to mintToken which allows user to mint multiple entities within one transaction to save gas and effort 

```solidity
202:  function mintWithBudget(
    ...
214:
215:@>     while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) { 
216:      _mintInternal(msg.sender, mintPrice);
      ...
225:  }
```

## [QA-06] amountMinted is not used for any purpose
### Impact
The variable `amountMinted` is incremented but never used, leading to wasted gas. This could have been avoided by emitting an event or providing some reference to the user.
### Instances
- Found in [contracts/TraitForgeNft/TraitForgeNft.sol at Line 217](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L217)

```solidity
202:  function mintWithBudget(
    ...
216:      _mintInternal(msg.sender, mintPrice);
217:@>       amountMinted++; 
218:      budgetLeft -= mintPrice;
      ...
225:  }
```