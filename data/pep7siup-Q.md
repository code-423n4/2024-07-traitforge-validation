## Quality Assurance Report

### Table of Contents

| Issue ID | Description                                               |
|----------|-----------------------------------------------------------|
| [QA-01](#qa-01-unbounded-loop-in-fetchlistings) | Unbounded loop in `fetchListings` causing DoS          |
| [QA-02](#qa-02-wrong-require-statement-for-maxslotindex-out-of-bound) | Wrong require statement for `maxSlotIndex` out-of-bounds |
| [QA-03](#qa-03-wrong-derivetokenparameters-reading-of-nukefactor) | Wrong `deriveTokenParameters` reading of `nukeFactor`  |
| [QA-04](#qa-04-semantically-wrong-whitelistendtime) | Semantically wrong `whitelistEndTime`                  |
| [QA-05](#qa-05-mintwithbudget-will-stop-working-after-the-first-generation) | `mintWithBudget` will stop working after the first generation |
| [QA-06](#qa-06-amountminted-is-not-used-for-any-purpose) | `amountMinted` is not used for any purpose             |

### [QA-01] Unbounded Loop in `fetchListings`

#### Impact

There are potentially thousands of listings that the loop has to process. Every time a listing of a certain `tokenId` is canceled and relisted with `listForForging`, the `listingCount` keeps increasing, potentially makes the loop exponentially large. Execution might fail due to exceeding the block size gas limit. Although view functions do not consume gas when called locally (e.g., using `eth_call`), they still need to execute within the block gas limit when run on an Ethereum node. If the `listingCount` is very high, memory allocation and processing might exceed the gas limit, causing the call to fail. External entities relying on `fetchListings` data could be rendered inoperable if a DoS occurs.

#### Instances

- Found in contracts/EntityForging/EntityForging.sol at [Line 50](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L50)

```solidity
48:  function fetchListings() external view returns (Listing[] memory _listings) {
49:    _listings = new Listing[](listingCount + 1);
50:@>     for (uint256 i = 1; i <= listingCount; ++i) { 
51:      _listings[i] = listings[i];
    ...
53:  }
```

### [QA-02] Wrong Require Statement for `maxSlotIndex` Out-of-Bounds

#### Impact

Out-of-Bounds access is invalid

#### Instances

- Found in contracts/EntropyGenerator/EntropyGenerator.sol at [Line 102](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L102)

> @>: currentSlotIndex == maxSlotIndex would cause out-of-bound indexing but still accepted in require check 

```solidity
101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {
102:@>     require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.'); 
103:    uint256 entropy = getEntropy(currentSlotIndex, currentNumberIndex);
...
120:  }
```

Fix:

```patch
-     require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.'); 
+     require(currentSlotIndex < maxSlotIndex, 'Max slot index reached.'); 
```

- Found in contracts/EntropyGenerator/EntropyGenerator.sol at [Line 168](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L168)

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

Fix:

```patch
-     require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');
+     require(slotIndex < maxSlotIndex, 'Slot index out of bounds.');
```

### [QA-03] Wrong `deriveTokenParameters` Reading of `nukeFactor`

#### Impact

The `nukeFactor` will always read as 0 due to the normalized `entropy` of max value being 999999, which is less than 4000000. This results in wrong value reading.

#### Instances

- Found in contracts/EntropyGenerator/EntropyGenerator.sol at [Line 152](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L152)
> @>: would always read as 0 due to entropy max value is 999999 less than 4000000 

```solidity
136:  function deriveTokenParameters(
    ...
149:     uint256 entropy = getEntropy(slotIndex, numberIndex);
...
151:    // example calculations using entropy to derive game-related parameters
152:@>     nukeFactor = entropy / 4000000; 
153:    forgePotential = getFirstDigit(entropy);
    ...
161:  }
```

### [QA-04] Semantically Wrong `whitelistEndTime`

#### Impact

`onlyWhitelisted` modifier is supposed to pass when block timestamp reach whitelistEndTime since the var name semantically depicts the whitelist priority period has officially ended. This causes Non-whitelist users timing for a mint race could fail for those willing to pay high gas to mint at `whitelistEndTime`. This results in a loss of funds and opportunity.

#### Instances

- Found in contracts/TraitForgeNft/TraitForgeNft.sol at [Line 52](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L52)

```solidity
51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
52:@>     if (block.timestamp <= whitelistEndTime) { 
53:      require(
        ...
59:  }
```

Fix:

```patch
-  if (block.timestamp <= whitelistEndTime) { 
+  if (block.timestamp < whitelistEndTime) { 
```

### [QA-05] `mintWithBudget` Will Stop Working After the First Generation

#### Impact

Breaks core functionality of the contract, making it unusable after the first generation is minted.

#### Instances

- Found in contracts/TraitForgeNft/TraitForgeNft.sol at [Line 215](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L215)
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

### [QA-06] `amountMinted` Is Not Used for Any Purpose

#### Impact

Wastes gas, as `amountMinted` is incremented but not used or referenced anywhere in the contract.

#### Instances

- Found in contracts/TraitForgeNft/TraitForgeNft.sol at [Line 217](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L217)
> @>: amountMinted increment every time new entity was minted but there's no reference to the variable. At least, there should emit an event including amountMinted or return it to user for references 

```solidity
202:  function mintWithBudget(
    ...
216:      _mintInternal(msg.sender, mintPrice);
217:@>       amountMinted++; 
218:      budgetLeft -= mintPrice;
      ...
225:  }
```