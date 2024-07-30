### Prevent re-setting a state variable with the same value
Not only is wasteful in terms of gas, but this is especially problematic when an event is emitted and the old and new values set are the same, as listeners might not expect this kind of scenario.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

69:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

101:    startPrice = _startPrice;	// @audit-issue

105:    priceIncrement = _priceIncrement;	// @audit-issue

111:    priceIncrementByGen = _priceIncrementByGen;	// @audit-issue

123:    rootHash = rootHash_;	// @audit-issue

127:    whitelistEndTime = endTime_;	// @audit-issue

294:    initialOwners[newItemId] = to;	// @audit-issue

326:    tokenEntropy[newTokenId] = entropy;	// @audit-issue

327:    tokenGenerations[newTokenId] = gen;	// @audit-issue

329:    initialOwners[newTokenId] = newOwner;	// @audit-issue
```
[69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L69-L69), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L101-L101), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L105-L105), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L111-L111), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L123-L123), [127](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L127-L127), [294](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L294-L294), [326](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L326-L326), [327](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L327-L327), [329](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L329-L329), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

33:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

37:    taxCut = _taxCut;	// @audit-issue

41:    oneYearInDays = value;	// @audit-issue

45:    minimumListFee = _fee;	// @audit-issue

96:    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L33-L33), [37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L37-L37), [41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L41-L41), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L45-L45), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L96-L96), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

30:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

34:    taxCut = _taxCut;	// @audit-issue

56:    listings[listingCount] = Listing(msg.sender, tokenId, price, true);	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L30-L30), [34](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L34-L34), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L56-L56), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

64:    taxCut = _taxCut;	// @audit-issue

68:    minimumDaysHeld = value;	// @audit-issue

72:    defaultNukeFactorIncrease = value;	// @audit-issue

76:    maxAllowedClaimDivisor = value;	// @audit-issue

80:    nukeFactorMaxParam = value;	// @audit-issue

85:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

90:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

95:    devAddress = account;	// @audit-issue

100:    daoAddress = account;	// @audit-issue

110:    ageMultiplier = _ageMultiplier;	// @audit-issue
```
[64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L64-L64), [68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L68-L68), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L72-L72), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L76-L76), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L80-L80), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L85-L85), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L90-L90), [95](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L95-L95), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L100-L100), [110](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L110-L110), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

37:    allowedCaller = _allowedCaller;	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L37-L37), 


#### Recommendation

Implement checks in your Solidity contracts to avoid re-setting state variables to their existing values. Prior to updating a state variable, compare the new value with the current value and proceed with the assignment only if they differ. Additionally, ensure that events related to state variable updates are emitted only when actual changes occur. This approach not only saves gas but also prevents confusion and unnecessary triggers in event listeners. Regularly reviewing and optimizing your contract for such redundancies can significantly enhance efficiency and clarity in contract operations.

### Superflous mint/burn event
Calling `_mint` and `_burn` emits an event, and so emitting a separate event in the same function is a waste of gas. Each event costs at least 375 gas. An additional 375 is paid for each indexed parameter.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

287:    _mint(to, newItemId);
288:    uint256 entropyValue = entropyGenerator.getNextEntropy();
289:
290:    tokenCreationTimestamps[newItemId] = block.timestamp;
291:    tokenEntropy[newItemId] = entropyValue;
292:    tokenGenerations[newItemId] = currentGeneration;
293:    generationMintCounts[currentGeneration]++;
294:    initialOwners[newItemId] = to;
295:
296:    if (!airdropContract.airdropStarted()) {
297:      airdropContract.addUserAmount(to, entropyValue);
298:    }
299:
300:    emit Minted(	// @audit-issue

323:    _mint(newOwner, newTokenId);
324:
325:    tokenCreationTimestamps[newTokenId] = block.timestamp;
326:    tokenEntropy[newTokenId] = entropy;
327:    tokenGenerations[newTokenId] = gen;
328:    generationMintCounts[gen]++;
329:    initialOwners[newTokenId] = newOwner;
330:
331:    if (
332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration
333:    ) {
334:      _incrementGeneration();
335:    }
336:
337:    if (!airdropContract.airdropStarted()) {
338:      airdropContract.addUserAmount(newOwner, entropy);
339:    }
340:
341:    emit NewEntityMinted(newOwner, newTokenId, gen, entropy);	// @audit-issue
```
[300](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L287-L300), [341](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L323-L341), 


#### Recommendation

Review your Solidity contracts, particularly those involving token operations, and remove any superfluous event emissions in functions that already call `_mint` or `_burn`. Rely on the events emitted by these standard functions, as they are designed to appropriately log minting and burning actions. By eliminating redundant event emissions, you can optimize the gas efficiency of your contract and streamline its operations. Ensure that any modifications maintain the necessary level of transparency and auditability required for your token's functionality and compliance needs.

### State Variable Access Within a Loop
State variable reads and writes are more expensive than local variable reads and writes. Therefore, it is recommended to replace state variable reads and writes within loops with a local variable. Gas savings should be multiplied by the average loop length.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

215:    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {	// @audit-issue: `_tokenIds` used in loop.

215:    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {	// @audit-issue: `maxTokensPerGen` used in loop.
```
[215](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L215-L215), [215](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L215-L215), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

51:      _listings[i] = listings[i];	// @audit-issue: `listings` used in loop.

50:    for (uint256 i = 1; i <= listingCount; ++i) {	// @audit-issue: `listingCount` used in loop.
```
[51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L51-L51), [50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L50-L50), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

57:        entropySlots[i] = pseudoRandomValue; // store the value in the slots array	// @audit-issue: `entropySlots` used in loop.

52:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue: `lastInitializedIndex` used in loop.

77:        entropySlots[i] = pseudoRandomValue;	// @audit-issue: `entropySlots` used in loop.

72:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue: `lastInitializedIndex` used in loop.

94:        entropySlots[i] = pseudoRandomValue;	// @audit-issue: `entropySlots` used in loop.

90:      for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {	// @audit-issue: `maxSlotIndex` used in loop.

90:      for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {	// @audit-issue: `lastInitializedIndex` used in loop.
```
[57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L57-L57), [52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L52-L52), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L77-L77), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L72-L72), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L94-L94), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L90-L90), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L90-L90), 


#### Recommendation

Optimize gas usage in Solidity by replacing state variable reads and writes within loops with local variable operations. This reduces gas costs significantly, especially when multiplied by the average loop length.

### Unnecessary Variable Initialization
In programming, it's a common practice to declare and initialize variables that will be used later in the code. However, if after declaration, the variable is not used anywhere else in the code, this leads to unnecessary variable initialization. In the context of Solidity and smart contracts, where gas efficiency is paramount, such redundant initializations not only consume extra gas but also clutter the codebase, making it less readable and harder to maintain.

Variables that are declared but never used represent dead code. They take up space in the bytecode, result in wasted gas when the contract is deployed, and can be confusing for anyone reading or auditing the code.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

212:    uint256 amountMinted = 0;	// @audit-issue
```
[212](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L212-L212), 


#### Recommendation

To avoid unnecessary variable initialization in Solidity code, regularly review and remove unused variables, use meaningful variable names, and avoid overly defensive coding. Prioritize gas efficiency and rigorous testing to maintain clean and efficient code.

### Unnecessary Assignment
The identified issue involves an unnecessary step of assigning a condition's result to a boolean variable before using it in an if-statement. This extra variable assignment is not required as the condition can be directly evaluated within the if-statement itself, making the code more concise and efficient.

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

92:    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy	// @audit-issue: This assignment is not needed. Binary operation can be used in condition directly.
93:    require(isForger, 'Only forgers can list for forging');
```
[92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L92-L93), 


#### Recommendation

Review your Solidity code for instances where boolean variables are unnecessarily assigned from conditions only to be used in immediate if-statements. Simplify these constructs by directly placing the conditional expression within the if-statement, eliminating the intermediary boolean variable. This practice not only streamlines your code, making it more concise, but also potentially reduces gas costs by minimizing the operations performed. Refactoring in this manner can enhance the readability and efficiency of your smart contracts. Always ensure that such optimizations maintain the original logic and intent of the code.

### `address(this)` should be cached
Cacheing saves gas when compared to repeating the calculation at each point it is used in the contract.

```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

53:    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract	// @audit-issue: `adress(this)` also used on line(s): [49, 48]
```
[53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L53-L53), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

160:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue: `adress(this)` also used on line(s): [159]
```
[160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L160-L160), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### `msg.value` should be cached
Cacheing saves gas when compared to repeating the calculation at each point it is used in the contract.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

195:    uint256 excessPayment = msg.value - mintPrice;	// @audit-issue: `msg.value` also used on line(s): [191]
```
[195](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L195-L195), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

17:      uint256 remaining = msg.value - (amountPerWeight * totalDevWeight);	// @audit-issue: `msg.value` also used on line(s): [27, 16, 24]
```
[17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L17-L17), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

66:      msg.value == listing.price,	// @audit-issue: `msg.value` also used on line(s): [89, 73, 72]
```
[66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L66-L66), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

42:    uint256 remainingFund = msg.value - devShare; // Calculate remaining funds to add to the fund	// @audit-issue: `msg.value` also used on line(s): [59, 41]
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L42-L42), 


#### Recommendation

Consider caching `msg.value` in a local variable at the start of payable functions where it is accessed multiple times. Implement this optimization by assigning `msg.value` to a local variable and using this variable for subsequent operations within the function. For example:
        ```solidity
        function myPayableFunction() public payable {
            uint256 cachedValue = msg.value;
            // Use cachedValue instead of msg.value for further operations
        }


### Unnecessary Event Emitting
Emitting two events in the same function with exactly the same event parameters is redundant and can lead to unnecessary gas consumption, as each `emit` operation adds to the transaction cost. These events can be merged into a single event to streamline contract operations and reduce gas usage, making the smart contract more efficient and cost-effective to interact with. This approach not only optimizes gas costs but also simplifies the contract's event logic for developers and applications integrating with the contract.

```solidity
Path: ./contracts/NukeFund/NukeFund.sol

56:      emit DevShareDistributed(devShare);	// @audit-issue: Exactly same parameters in event emitting also used on line(s): `[49]`
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L56-L56), 


#### Recommendation

Your code can be optimized with changing the usage:
```solidity
emit SaleEnded(timestamp);
emit ClaimStarted(timestamp);
```
To
```solidity
emit SaleEndedAndClaimStarted(timestamp); // Declare general one and remove old ones.
```


### Revert String Size Optimization
Shortening the revert strings to fit within 32 bytes will decrease deployment time and decrease runtime Gas when the revert condition is met.

Revert strings that are longer than 32 bytes require at least one additional `mstore`, along with additional overhead to calculate memory offset, etc.



```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

115:    require(	// @audit-issue: String length is `35`
116:      maxGeneration_ >= currentGeneration,
117:      "can't below than current generation"
118:    );

142:    require(	// @audit-issue: String length is `45`
143:      isApprovedOrOwner(msg.sender, tokenId),
144:      'ERC721: caller is not token owner or approved'
145:    );

191:    require(msg.value >= mintPrice, 'Insufficient ETH send for minting.');	// @audit-issue: String length is `34`

235:    require(	// @audit-issue: String length is `35`
236:      ownerOf(tokenId) != address(0),
237:      'ERC721: query for nonexistent token'
238:    );

257:    require(	// @audit-issue: String length is `35`
258:      ownerOf(tokenId) != address(0),
259:      'ERC721: query for nonexistent token'
260:    );

267:    require(	// @audit-issue: String length is `35`
268:      ownerOf(tokenId) != address(0),
269:      'ERC721: query for nonexistent token'
270:    );

394:    require(!paused(), 'ERC721Pausable: token transfer while paused');	// @audit-issue: String length is `43`
```
[115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L115-L118), [142](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L142-L145), [191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L191-L191), [235](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L235-L238), [257](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L257-L260), [267](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L267-L270), [394](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L394-L394), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

73:    require(!_listingInfo.isListed, 'Token is already listed for forging');	// @audit-issue: String length is `35`

78:    require(	// @audit-issue: String length is `45`
79:      fee >= minimumListFee,
80:      'Fee should be higher than minimum listing fee'
81:    );

87:    require(	// @audit-issue: String length is `36`
88:      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
89:      'Entity has reached its forging limit'
90:    );

93:    require(isForger, 'Only forgers can list for forging');	// @audit-issue: String length is `33`

107:    require(	// @audit-issue: String length is `38`
108:      _forgerListingInfo.isListed,
109:      "Forger's entity not listed for forging"
110:    );

115:    require(	// @audit-issue: String length is `50`
116:      nftContract.ownerOf(forgerTokenId) != msg.sender,
117:      'Caller should be different from forger token owner'
118:    );
```
[73](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L73-L73), [78](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L78-L81), [87](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L87-L90), [93](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L93-L93), [107](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L107-L110), [115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L115-L118), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

47:    require(	// @audit-issue: String length is `46`
48:      nftContract.getApproved(tokenId) == address(this) ||
49:        nftContract.isApprovedForAll(msg.sender, address(this)),
50:      'Contract must be approved to transfer the NFT.'
51:    );

65:    require(	// @audit-issue: String length is `41`
66:      msg.value == listing.price,
67:      'ETH sent does not match the listing price'
68:    );

98:    require(	// @audit-issue: String length is `38`
99:      listing.seller == msg.sender,
100:      'Only the seller can canel the listing.'
101:    );
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L47-L51), [65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L65-L68), [98](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L98-L101), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

137:    require(	// @audit-issue: String length is `44`
138:      nftContract.ownerOf(tokenId) != address(0),
139:      'ERC721: operator query for nonexistent token'
140:    );

154:    require(	// @audit-issue: String length is `45`
155:      nftContract.isApprovedOrOwner(msg.sender, tokenId),
156:      'ERC721: caller is not token owner or approved'
157:    );

158:    require(	// @audit-issue: String length is `46`
159:      nftContract.getApproved(tokenId) == address(this) ||
160:        nftContract.isApprovedForAll(msg.sender, address(this)),
161:      'Contract must be approved to transfer the NFT.'
162:    );

186:    require(	// @audit-issue: String length is `44`
187:      nftContract.ownerOf(tokenId) != address(0),
188:      'ERC721: operator query for nonexistent token'
189:    );
```
[137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L137-L140), [154](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L154-L157), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L158-L162), [186](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L186-L189), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

65:    require(	// @audit-issue: String length is `41`
66:      lastInitializedIndex >= batchSize1 && lastInitializedIndex < batchSize2,
67:      'Batch 2 not ready or already initialized.'
68:    );

85:    require(	// @audit-issue: String length is `39`
86:      lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,
87:      'Batch 3 not ready or already completed.'
88:    );
```
[65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L65-L68), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L85-L88), 


#### Recommendation


To optimize Gas usage in your Solidity contract, it is recommended to keep revert strings as short as possible and to ensure that they fit within 32 bytes. It is possible to use abbreviations or simplified error messages to keep the string length short. Doing so can reduce the amount of Gas used during deployment and runtime when the revert condition is met.


### Custom Errors in Solidity for Gas Efficiency
Starting from Solidity version 0.8.4, the language introduced a feature known as "custom errors". These custom errors provide a way for developers to define more descriptive and semantically meaningful error conditions without relying on string messages. Prior to this version, developers often used the `require` statement with string error messages to handle specific conditions or validations. However, every unique string used as a revert reason consumes gas, making transactions more expensive.

Custom errors, on the other hand, are identified by their name and the types of their parameters only, and they do not have the overhead of string storage. This means that, when using custom errors instead of `require` statements with string messages, the gas consumption can be significantly reduced, leading to more gas-efficient contracts.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

53:      require(
54:        MerkleProof.verify(proof, rootHash, leaf),
55:        'Not whitelisted user'
56:      );

77:    require(entityForgingAddress_ != address(0), 'Invalid address');	// @audit-issue

85:    require(entropyGeneratorAddress_ != address(0), 'Invalid address');	// @audit-issue

91:    require(airdrop_ != address(0), 'Invalid address');	// @audit-issue

115:    require(	// @audit-issue
116:      maxGeneration_ >= currentGeneration,
117:      "can't below than current generation"
118:    );

142:    require(	// @audit-issue
143:      isApprovedOrOwner(msg.sender, tokenId),
144:      'ERC721: caller is not token owner or approved'
145:    );

159:    require(	// @audit-issue
160:      msg.sender == address(entityForgingContract),
161:      'unauthorized caller'
162:    );

166:    require(newGeneration <= maxGeneration, "can't be over max generation");	// @audit-issue

191:    require(msg.value >= mintPrice, 'Insufficient ETH send for minting.');	// @audit-issue

198:      require(refundSuccess, 'Refund of excess payment failed.');

223:      require(refundSuccess, 'Refund failed.');

235:    require(	// @audit-issue
236:      ownerOf(tokenId) != address(0),
237:      'ERC721: query for nonexistent token'
238:    );

257:    require(	// @audit-issue
258:      ownerOf(tokenId) != address(0),
259:      'ERC721: query for nonexistent token'
260:    );

267:    require(	// @audit-issue
268:      ownerOf(tokenId) != address(0),
269:      'ERC721: query for nonexistent token'
270:    );

316:    require(	// @audit-issue
317:      generationMintCounts[gen] < maxTokensPerGen,
318:      'Exceeds maxTokensPerGen'
319:    );

346:    require(	// @audit-issue
347:      generationMintCounts[currentGeneration] >= maxTokensPerGen,
348:      'Generation limit not yet reached'
349:    );

359:    require(address(this).balance >= totalAmount, 'Insufficient balance');	// @audit-issue

362:    require(success, 'ETH send failed');	// @audit-issue

394:    require(!paused(), 'ERC721Pausable: token transfer while paused');	// @audit-issue
```
[52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L53-L56), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L77-L77), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L85-L85), [91](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L91-L91), [115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L115-L118), [142](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L142-L145), [159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L159-L162), [166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L166-L166), [191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L191-L191), [196](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L198-L198), [221](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L223-L223), [235](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L235-L238), [257](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L257-L260), [267](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L267-L270), [316](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L316-L319), [346](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L346-L349), [359](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L359-L359), [362](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L362-L362), [394](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L394-L394), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

73:    require(!_listingInfo.isListed, 'Token is already listed for forging');	// @audit-issue

74:    require(	// @audit-issue
75:      nftContract.ownerOf(tokenId) == msg.sender,
76:      'Caller must own the token'
77:    );

78:    require(	// @audit-issue
79:      fee >= minimumListFee,
80:      'Fee should be higher than minimum listing fee'
81:    );

87:    require(	// @audit-issue
88:      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
89:      'Entity has reached its forging limit'
90:    );

93:    require(isForger, 'Only forgers can list for forging');	// @audit-issue

107:    require(	// @audit-issue
108:      _forgerListingInfo.isListed,
109:      "Forger's entity not listed for forging"
110:    );

111:    require(	// @audit-issue
112:      nftContract.ownerOf(mergerTokenId) == msg.sender,
113:      'Caller must own the merger token'
114:    );

115:    require(	// @audit-issue
116:      nftContract.ownerOf(forgerTokenId) != msg.sender,
117:      'Caller should be different from forger token owner'
118:    );

119:    require(	// @audit-issue
120:      nftContract.getTokenGeneration(mergerTokenId) ==
121:        nftContract.getTokenGeneration(forgerTokenId),
122:      'Invalid token generation'
123:    );

126:    require(msg.value >= forgingFee, 'Insufficient fee for forging');	// @audit-issue

137:    require(mergerEntropy % 3 != 0, 'Not merger');	// @audit-issue

140:    require(	// @audit-issue
141:      mergerForgePotential > 0 &&
142:        forgingCounts[mergerTokenId] <= mergerForgePotential,
143:      'forgePotential insufficient'
144:    );

157:    require(success, 'Failed to send to NukeFund');	// @audit-issue

159:    require(success_forge, 'Failed to send to Forge Owner');	// @audit-issue

180:    require(	// @audit-issue
181:      nftContract.ownerOf(tokenId) == msg.sender ||
182:        msg.sender == address(nftContract),
183:      'Caller must own the token'
184:    );

185:    require(	// @audit-issue
186:      listings[listedTokenIds[tokenId]].isListed,
187:      'Token not listed for forging'
188:    );
```
[73](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L73-L73), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L74-L77), [78](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L78-L81), [87](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L87-L90), [93](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L93-L93), [107](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L107-L110), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L111-L114), [115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L115-L118), [119](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L119-L123), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L126-L126), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L137-L137), [140](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L140-L144), [157](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L157-L157), [159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L159-L159), [180](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L180-L184), [185](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L185-L188), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

21:        require(success, 'Failed to send Ether to owner');

25:      require(success, 'Failed to send Ether to owner');

32:    require(weight > 0, 'Invalid weight');	// @audit-issue

33:    require(info.weight == 0, 'Already registered');	// @audit-issue

42:    require(weight > 0, 'Invalid weight');	// @audit-issue

43:    require(info.weight > 0, 'Not dev address');	// @audit-issue

53:    require(info.weight > 0, 'Not dev address');	// @audit-issue

90:    require(success, 'Failed to send Reward');	// @audit-issue
```
[15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L21-L21), [15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L25-L25), [32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L32-L32), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L33-L33), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L43-L43), [53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L53-L53), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L90-L90), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

42:    require(price > 0, 'Price must be greater than zero');	// @audit-issue

43:    require(	// @audit-issue
44:      nftContract.ownerOf(tokenId) == msg.sender,
45:      'Sender must be the NFT owner.'
46:    );

47:    require(	// @audit-issue
48:      nftContract.getApproved(tokenId) == address(this) ||
49:        nftContract.isApprovedForAll(msg.sender, address(this)),
50:      'Contract must be approved to transfer the NFT.'
51:    );

65:    require(	// @audit-issue
66:      msg.value == listing.price,
67:      'ETH sent does not match the listing price'
68:    );

69:    require(listing.seller != address(0), 'NFT is not listed for sale.');	// @audit-issue

80:    require(success, 'Failed to send to seller');	// @audit-issue

98:    require(	// @audit-issue
99:      listing.seller == msg.sender,
100:      'Only the seller can canel the listing.'
101:    );

102:    require(listing.isActive, 'Listing is not active.');	// @audit-issue

113:    require(nukeFundAddress != address(0), 'NukeFund address not set');	// @audit-issue

115:    require(success, 'Failed to send Ether to NukeFund');	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L43-L46), [47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L47-L51), [65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L65-L68), [69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L69-L69), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L80-L80), [98](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L98-L101), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L102-L102), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L113-L113), [115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L115-L115), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

48:      require(success, 'ETH send failed');

52:      require(success, 'ETH send failed');

55:      require(success, 'ETH send failed');

119:    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');	// @audit-issue

137:    require(	// @audit-issue
138:      nftContract.ownerOf(tokenId) != address(0),
139:      'ERC721: operator query for nonexistent token'
140:    );

154:    require(	// @audit-issue
155:      nftContract.isApprovedOrOwner(msg.sender, tokenId),
156:      'ERC721: caller is not token owner or approved'
157:    );

158:    require(	// @audit-issue
159:      nftContract.getApproved(tokenId) == address(this) ||
160:        nftContract.isApprovedForAll(msg.sender, address(this)),
161:      'Contract must be approved to transfer the NFT.'
162:    );

163:    require(canTokenBeNuked(tokenId), 'Token is not mature yet');	// @audit-issue

178:    require(success, 'Failed to send Ether');	// @audit-issue

186:    require(	// @audit-issue
187:      nftContract.ownerOf(tokenId) != address(0),
188:      'ERC721: operator query for nonexistent token'
189:    );
```
[46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L48-L48), [46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L52-L52), [46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L55-L55), [119](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L119-L119), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L137-L140), [154](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L154-L157), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L158-L162), [163](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L163-L163), [178](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L178-L178), [186](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L186-L189), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

26:    require(msg.sender == allowedCaller, 'Caller is not allowed');	// @audit-issue

48:    require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');	// @audit-issue

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');

65:    require(	// @audit-issue
66:      lastInitializedIndex >= batchSize1 && lastInitializedIndex < batchSize2,
67:      'Batch 2 not ready or already initialized.'
68:    );

76:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');

85:    require(	// @audit-issue
86:      lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,
87:      'Batch 3 not ready or already completed.'
88:    );

102:    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');	// @audit-issue

168:    require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');	// @audit-issue

178:    require(position <= 72, 'Position calculation error');	// @audit-issue
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L26-L26), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L48-L48), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), [65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L65-L68), [71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L76-L76), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L85-L88), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L102-L102), [168](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L168-L168), [178](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L178-L178), 


#### Recommendation


It is recommended to use custom errors instead of revert strings to reduce gas costs, especially during contract deployment. Custom errors can be defined using the error keyword and can include dynamic information.

### Using Prefix Operators Costs Less Gas Than Postfix Operators in Loops
Conditions can be optimized issues in Solidity refer to situations where smart contract developers write conditional statements that can be simplified or optimized for better gas efficiency, readability, and maintainability. Optimizing conditions can lead to more cost-effective and secure smart contracts.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

217:      amountMinted++;	// @audit-issue

285:    _tokenIds++;	// @audit-issue

293:    generationMintCounts[currentGeneration]++;	// @audit-issue

321:    _tokenIds++;	// @audit-issue

328:    generationMintCounts[gen]++;	// @audit-issue

350:    currentGeneration++;	// @audit-issue
```
[217](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L217-L217), [285](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L285-L285), [293](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L293-L293), [321](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L321-L321), [328](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L328-L328), [350](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L350-L350), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

133:    forgingCounts[forgerTokenId]++;	// @audit-issue

139:    forgingCounts[mergerTokenId]++;	// @audit-issue
```
[133](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L133-L133), [139](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L139-L139), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

52:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue

72:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue

90:      for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {	// @audit-issue

110:        currentSlotIndex++;	// @audit-issue

113:      currentNumberIndex++;	// @audit-issue

192:      digits++;	// @audit-issue
```
[52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L52-L52), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L72-L72), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L90-L90), [110](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L110-L110), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L113-L113), [192](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L192-L192), 


#### Recommendation

To improve gas efficiency in your Solidity code, prefer using prefix operators (e.g., `++i` or `--i`) instead of postfix operators (e.g., `i++` or `i--`) within loops. Prefix operators typically result in lower gas costs and can contribute to more efficient contract execution.

### Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration	// @audit-issue: Same index access of variable `generationMintCounts` is used also on line(s): ['317'].
```
[332](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L332-L332), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

203:    } else if (block.timestamp >= lastForgeResetTimestamp[tokenId] + oneYear) {	// @audit-issue: Same index access of variable `lastForgeResetTimestamp` is used also on line(s): ['201'].
```
[203](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L203-L203), 


#### Recommendation

When a mapping or array value is accessed multiple times within a function, cache it in a local `storage` or `calldata` variable. This approach minimizes the gas cost by reducing the number of hash computations for mappings and offset calculations for arrays. Carefully review your functions to identify opportunities for such optimizations, especially in functions with frequent or repeated accesses to the same mapping or array element, to enhance gas efficiency in your contracts.

### State Variables That Are Used Multiple Times In a Function Should Be Cached In Stack Variables
When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. Saves 100 gas per instance.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

303:      currentGeneration,	// @audit-issue: State variable `currentGeneration` is used also on line(s): ['292', '293', '281'].

332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration	// @audit-issue: State variable `maxTokensPerGen` is used also on line(s): ['317'].

332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration	// @audit-issue: State variable `generationMintCounts` is used also on line(s): ['317'].

351:    generationMintCounts[currentGeneration] = 0;	// @audit-issue: State variable `currentGeneration` is used also on line(s): ['354', '347'].
```
[303](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L303-L303), [332](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L332-L332), [332](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L332-L332), [351](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L351-L351), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

50:    for (uint256 i = 1; i <= listingCount; ++i) {	// @audit-issue: State variable `listingCount` is used also on line(s): ['49'].

97:    listedTokenIds[tokenId] = listingCount;	// @audit-issue: State variable `listingCount` is used also on line(s): ['96'].

203:    } else if (block.timestamp >= lastForgeResetTimestamp[tokenId] + oneYear) {	// @audit-issue: State variable `lastForgeResetTimestamp` is used also on line(s): ['201'].
```
[50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L50-L50), [97](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L97-L97), [203](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L203-L203), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

15:    if (totalDevWeight > 0) {	// @audit-issue: State variable `totalDevWeight` is used also on line(s): ['17', '16'].

45:    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;	// @audit-issue: State variable `totalRewardDebt` is used also on line(s): ['46'].

43:    require(info.weight > 0, 'Not dev address');	// @audit-issue: State variable `info` is used also on line(s): ['44', '45'].

56:    info.rewardDebt = totalRewardDebt;	// @audit-issue: State variable `totalRewardDebt` is used also on line(s): ['55'].

54:    totalDevWeight -= info.weight;	// @audit-issue: State variable `info` is used also on line(s): ['53', '55'].

65:      (totalRewardDebt - info.rewardDebt) *	// @audit-issue: State variable `totalRewardDebt` is used also on line(s): ['74'].
```
[15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L15-L15), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L45-L45), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L43-L43), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L56-L56), [54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L54-L54), [65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L65-L65), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

57:    listedTokenIds[tokenId] = listingCount;	// @audit-issue: State variable `listingCount` is used also on line(s): ['56'].

83:    delete listings[listedTokenIds[tokenId]]; // remove listing	// @audit-issue: State variable `listedTokenIds` is used also on line(s): ['64'].

106:    delete listings[listedTokenIds[tokenId]]; // mark the listing as inactive or delete it	// @audit-issue: State variable `listedTokenIds` is used also on line(s): ['95'].
```
[57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L57-L57), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L83-L83), [106](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L106-L106), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

167:    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size	// @audit-issue: State variable `fund` is used also on line(s): ['181', '166'].
```
[167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L167-L167), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

48:    require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');	// @audit-issue: State variable `lastInitializedIndex` is used also on line(s): ['50', '52'].

48:    require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');	// @audit-issue: State variable `batchSize1` is used also on line(s): ['50'].

66:      lastInitializedIndex >= batchSize1 && lastInitializedIndex < batchSize2,	// @audit-issue: State variable `lastInitializedIndex` is used also on line(s): ['70', '66', '72'].

70:    uint256 endIndex = lastInitializedIndex + batchSize1;	// @audit-issue: State variable `batchSize1` is used also on line(s): ['66'].

86:      lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,	// @audit-issue: State variable `lastInitializedIndex` is used also on line(s): ['86', '90'].

90:      for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {	// @audit-issue: State variable `maxSlotIndex` is used also on line(s): ['86', '97'].

102:    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');	// @audit-issue: State variable `currentSlotIndex` is used also on line(s): ['107', '103'].

107:      if (currentSlotIndex >= maxSlotIndex - 1) {	// @audit-issue: State variable `maxSlotIndex` is used also on line(s): ['102'].

105:    if (currentNumberIndex >= maxNumberIndex - 1) {	// @audit-issue: State variable `currentNumberIndex` is used also on line(s): ['103'].
```
[48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L48-L48), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L48-L48), [66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L66-L66), [70](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L70-L70), [86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L86-L86), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L90-L90), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L102-L102), [107](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L107-L107), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L105-L105), 


#### Recommendation

Cache state variables in stack or local memory variables within functions when they are used multiple times. This approach replaces costlier Gwarmaccess operations with cheaper stack reads, saving approximately 100 gas per instance and optimizing overall contract performance.

### Missing Constant Modifier For State Variables
State variables that are never re-assigned after their initial assignment should typically be declared with the `constant` modifier. This ensures that these variables remain unmodified and communicates their unchanging nature. Using the `constant` modifier also offers potential gas savings, as accessing these variables does not require any storage operations.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

22:  uint256 public maxTokensPerGen = 10000;	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L22-L22), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

14:  uint256 private batchSize1 = 256;	// @audit-issue

15:  uint256 private batchSize2 = 512;	// @audit-issue

17:  uint256 private maxSlotIndex = 770;	// @audit-issue

18:  uint256 private maxNumberIndex = 13;	// @audit-issue
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L14-L14), [15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L15-L15), [17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L17-L17), [18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L18-L18), 


#### Recommendation

Declare state variables that are not reassigned after initialization with the 'constant' modifier in Solidity. This clarifies their unchanging nature, improves code readability, and saves gas by avoiding storage operations when accessing these variables.

### State Variables Only Set in The Constructor Should Be Declared `immutable`
Avoids a Gsset(** 20000 gas**) in the constructor, and replaces the first access in each transaction(Gcoldsload - ** 2100 gas **) and each access thereafter(Gwarmacces - ** 100 gas **) with a`PUSH32`(** 3 gas **).

While`string`s are not value types, and therefore cannot be`immutable` / `constant` if not hard - coded outside of the constructor, the same behavior can be achieved by making the current contract `abstract` with `virtual` functions for the`string` accessors, and having a child contract override the functions with the hard - coded implementation - specific values.

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

11:  ITraitForgeNft public nftContract;	// @audit-issue
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L11-L11), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

12:  ITraitForgeNft public nftContract;	// @audit-issue
```
[12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L12-L12), 


#### Recommendation

Consider marking state variables as an immutable that never changes on the contract.

### Increments can be `unchecked` in for-loops
Newer versions of the Solidity compiler will check for integer overflows and underflows automatically. This provides safety but increases gas costs.
When an unsigned integer is guaranteed to never overflow, the unchecked feature of Solidity can be used to save gas costs.A common case for this is for-loops using a strictly-less-than comparision in their conditional statement.

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

50:    for (uint256 i = 1; i <= listingCount; ++i) {	// @audit-issue
```
[50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L50-L50), 


#### Recommendation

Use unchecked math to block overflow / underflow check to save Gas.

### Divisions can be unchecked to save gas
The expression type(int).min/(-1) is the only case where division causes an overflow. Therefore, uncheck can be used to save gas in scenarios where it is certain that such an overflow will not occur.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

173:    uint256 newEntropy = (forgerEntropy + mergerEntropy) / 2;	// @audit-issue
```
[173](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L173-L173), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

86:    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue

138:    uint8 mergerForgePotential = uint8((mergerEntropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue

146:    uint256 devFee = forgingFee / taxCut;	// @audit-issue
```
[86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L86-L86), [138](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L138-L138), [146](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L146-L146), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

16:      uint256 amountPerWeight = msg.value / totalDevWeight;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L16-L16), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

72:    uint256 nukeFundContribution = msg.value / taxCut;	// @audit-issue
```
[72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L72-L72), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

41:    uint256 devShare = msg.value / taxCut; // Calculate developer's share (10%)	// @audit-issue

121:    uint256 daysOld = (block.timestamp -	// @audit-issue

128:    uint256 age = (daysOld *	// @audit-issue

145:    uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits	// @audit-issue

147:    uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) /	// @audit-issue

166:    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; // Calculate the potential claim amount based on the finalNukeFactor	// @audit-issue

167:    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size	// @audit-issue
```
[41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L41-L41), [121](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L121-L121), [128](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L128-L128), [145](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L145-L145), [147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L147-L147), [166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L166-L166), [167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L167-L167), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

152:    nukeFactor = entropy / 4000000;	// @audit-issue

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue
```
[152](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L152-L152), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), 


#### Recommendation

Utilize 'unchecked' blocks in Solidity for divisions where overflow is impossible, such as when 'type(int).min/(-1)' is not a concern. This can save gas by bypassing overflow checks in these specific cases.

### Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
Unchecked keyword can be added to such scenerios: 
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

195:    uint256 excessPayment = msg.value - mintPrice;	// @audit-issue
```
[195](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L195-L195), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue
```
[181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), 


#### Recommendation

In scenarios where subtraction cannot result in underflow due to prior `require()` or `if`-statements, wrap these operations in an `unchecked` block to save gas. This optimization should only be applied when the safety of the operation is assured. Carefully analyze each case to confirm that underflow is impossible before implementing `unchecked` blocks, as incorrect usage can lead to vulnerabilities in the contract.

### Stack variable is only used once
If the variable is only accessed once, it's cheaper to use the assigned value directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

147:      uint256 entropy = getTokenEntropy(tokenId);	// @audit-issue: entropy used only on line: 148

169:    (uint256 forgerEntropy, uint256 mergerEntropy) = getEntropiesForTokens(	// @audit-issue: forgerEntropy used only on line: 173
170:      parent1Id,
171:      parent2Id
172:    );

169:    (uint256 forgerEntropy, uint256 mergerEntropy) = getEntropiesForTokens(	// @audit-issue: mergerEntropy used only on line: 173
170:      parent1Id,
171:      parent2Id
172:    );

173:    uint256 newEntropy = (forgerEntropy + mergerEntropy) / 2;	// @audit-issue: newEntropy used only on line: 176

176:    uint256 newTokenId = _mintNewEntity(newOwner, newEntropy, newGeneration);	// @audit-issue: newTokenId used only on line: 178

197:      (bool refundSuccess, ) = msg.sender.call{ value: excessPayment }('');	// @audit-issue: refundSuccess used only on line: 198

222:      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');	// @audit-issue: refundSuccess used only on line: 223

228:    uint256 currentGenMintCount = generationMintCounts[currentGeneration];	// @audit-issue: currentGenMintCount used only on line: 229

229:    uint256 priceIncrease = priceIncrement * currentGenMintCount;	// @audit-issue: priceIncrease used only on line: 230

230:    uint256 price = startPrice + priceIncrease;	// @audit-issue: price used only on line: 231

275:    uint256 entropy = tokenEntropy[tokenId];	// @audit-issue: entropy used only on line: 276

276:    uint256 roleIndicator = entropy % 3;	// @audit-issue: roleIndicator used only on line: 277

361:    (bool success, ) = nukeFundAddress.call{ value: totalAmount }('');	// @audit-issue: success used only on line: 362
```
[147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L147-L147), [169](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L169-L172), [169](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L169-L172), [173](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L173-L173), [176](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L176-L176), [197](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L197-L197), [222](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L222-L222), [228](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L228-L228), [229](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L229-L229), [230](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L230-L230), [275](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L275-L275), [276](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L276-L276), [361](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L361-L361), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

71:    Listing memory _listingInfo = listings[listedTokenIds[tokenId]];	// @audit-issue: _listingInfo used only on line: 73

92:    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy	// @audit-issue: isForger used only on line: 93

147:    uint256 forgerShare = forgingFee - devFee;	// @audit-issue: forgerShare used only on line: 158

148:    address payable forgerOwner = payable(nftContract.ownerOf(forgerTokenId));	// @audit-issue: forgerOwner used only on line: 158

156:    (bool success, ) = nukeFundAddress.call{ value: devFee }('');	// @audit-issue: success used only on line: 157

158:    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');	// @audit-issue: success_forge used only on line: 159

164:    uint256 newEntropy = nftContract.getTokenEntropy(newTokenId);	// @audit-issue: newEntropy used only on line: 170

200:    uint256 oneYear = oneYearInDays;	// @audit-issue: oneYear used only on line: 203
```
[71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L71-L71), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L92-L92), [147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L147-L147), [148](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L148-L148), [156](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L156-L156), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L158-L158), [164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L164-L164), [200](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L200-L200), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

89:    (bool success, ) = payable(to).call{ value: amount }('');	// @audit-issue: success used only on line: 90
```
[89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L89-L89), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

73:    uint256 sellerProceeds = msg.value - nukeFundContribution;	// @audit-issue: sellerProceeds used only on line: 77

77:    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(	// @audit-issue: success used only on line: 80
78:      ''
79:    );

114:    (bool success, ) = nukeFundAddress.call{ value: amount }('');	// @audit-issue: success used only on line: 115
```
[73](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L73-L73), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L77-L79), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L114-L114), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

42:    uint256 remainingFund = msg.value - devShare; // Calculate remaining funds to add to the fund	// @audit-issue: remainingFund used only on line: 44

121:    uint256 daysOld = (block.timestamp -	// @audit-issue: daysOld used only on line: 128
122:      nftContract.getTokenCreationTimestamp(tokenId)) /
123:      60 /
124:      60 /
125:      24;

126:    uint256 perfomanceFactor = nftContract.getTokenEntropy(tokenId) % 10;	// @audit-issue: perfomanceFactor used only on line: 129

128:    uint256 age = (daysOld *	// @audit-issue: age used only on line: 132
129:      perfomanceFactor *
130:      MAX_DENOMINATOR *
131:      ageMultiplier) / 365; // add 5 digits for decimals

142:    uint256 entropy = nftContract.getTokenEntropy(tokenId);	// @audit-issue: entropy used only on line: 145

143:    uint256 adjustedAge = calculateAge(tokenId);	// @audit-issue: adjustedAge used only on line: 147

145:    uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits	// @audit-issue: initialNukeFactor used only on line: 148

147:    uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) /	// @audit-issue: finalNukeFactor used only on line: 150
148:      MAX_DENOMINATOR) + initialNukeFactor;

166:    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; // Calculate the potential claim amount based on the finalNukeFactor	// @audit-issue: potentialClaimAmount used only on line: 172

167:    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size	// @audit-issue: maxAllowedClaimAmount used only on line: 171

177:    (bool success, ) = payable(msg.sender).call{ value: claimAmount }('');	// @audit-issue: success used only on line: 178

190:    uint256 tokenAgeInSeconds = block.timestamp -	// @audit-issue: tokenAgeInSeconds used only on line: 193
191:      nftContract.getTokenLastTransferredTimestamp(tokenId);
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L42-L42), [121](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L121-L125), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L126-L126), [128](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L128-L131), [142](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L142-L142), [143](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L143-L143), [145](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L145-L145), [147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L147-L148), [166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L166-L166), [167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L167-L167), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L177-L177), [190](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L190-L191), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

157:    uint256 role = entropy % 3;	// @audit-issue: role used only on line: 158

180:    uint256 slotValue = entropySlots[slotIndex]; // slice the required [art of the entropy value	// @audit-issue: slotValue used only on line: 181

182:    uint256 paddedEntropy = entropy * (10 ** (6 - numberOfDigits(entropy)));	// @audit-issue: paddedEntropy used only on line: 184

211:    uint256 slotIndexSelection = (hashValue % 258) + 512;	// @audit-issue: slotIndexSelection used only on line: 214

212:    uint256 numberIndexSelection = hashValue % 13;	// @audit-issue: numberIndexSelection used only on line: 215
```
[157](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L157-L157), [180](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L180-L180), [182](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L182-L182), [211](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L211-L211), [212](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L212-L212), 


#### Recommendation

Eliminate single-use stack variables in Solidity to optimize gas consumption. Directly use the assigned value in the place of the variable. This approach saves the 3 gas typically used for the extra stack assignment, streamlining the function's execution and enhancing overall gas efficiency.

### Avoid Using State Variables Directly in `emit` for Gas Efficiency
In Solidity, emitting events is a common way to log contract activity and changes, especially for off-chain monitoring and interfacing. However, using state variables directly in `emit` statements can lead to increased gas costs. Each access to a state variable incurs gas due to storage reading operations. When these variables are used directly in `emit` statements, especially within functions that perform multiple operations, the cumulative gas cost can become significant. Instead, caching state variables in memory and using these local copies in `emit` statements can optimize gas usage.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

300:    emit Minted(
301:      msg.sender,
302:      newItemId,
303:      currentGeneration,	// @audit-issue: `currentGeneration` is a state variable and used on line(s): ['292', '293', '281']
304:      entropyValue,
305:      mintPrice
306:    );

354:    emit GenerationIncremented(currentGeneration);	// @audit-issue: `currentGeneration` is a state variable and used on line(s): ['351', '347']

364:    emit FundsDistributedToNukeFund(nukeFundAddress, totalAmount);	// @audit-issue: `nukeFundAddress` is a state variable and used on line(s): ['361']
```
[303](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L300-L306), [354](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L354-L354), [364](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L364-L364), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

181:    emit FundBalanceUpdated(fund); // Update the fund balance	// @audit-issue: `fund` is a state variable and used on line(s): ['167', '166']
```
[181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L181-L181), 


#### Recommendation


To optimize gas efficiency, cache state variables in memory when they are used multiple times within a function, including in `emit` statements.

### `x += y` costs less gas than `x = x + y` for state variables
In Solidity, optimizing gas is vital for efficient smart contracts. The compound assignment `x += y` has been proven to consume less gas compared to `x = x + y` when updating state variables. This approach is not only gas-efficient but also aligns with best coding practices, offering a cleaner, more readable code. It is also valid for operations including: `-`, `/`, `*` , `&`, `|` , `^`, `%`

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

352:    priceIncrement = priceIncrement + priceIncrementByGen;	// @audit-issue: `priceIncrement` can be assigned as `priceIncrement += ..`
```
[352](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L352-L352), 


#### Recommendation

Use Compound Assignments: Adopt `x += y` for concise, gas-efficient, and easy-to-read code. This approach is recognized as a best practice in Solidity development.

### `Internal` functions only called once can be inlined to save gas
If an internal function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex. Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./contracts/DevFund/DevFund.sol

83:  function safeRewardTransfer(	// @audit-issue
```
[83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L83), 


#### Recommendation

Inline 'internal' functions in Solidity that are called only once to save gas. This avoids the additional gas cost of 20 to 40 units associated with extra JUMP instructions and stack operations required for separate function calls, unless the calling function becomes too complex.

### `Private` functions only called once can be inlined to save gas
If a private function is only used once, there is no need to modularize it, unless the function calling it would otherwise be too long and complex. Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

311:  function _mintNewEntity(	// @audit-issue

358:  function _distributeFunds(uint256 totalAmount) private {	// @audit-issue
```
[311](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L311-L311), [358](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L358-L358), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

112:  function transferToNukeFund(uint256 amount) private {	// @audit-issue
```
[112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L112-L112), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue
```
[188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), 


#### Recommendation

Inline 'private' functions in Solidity that are called only once to save gas. This avoids the additional gas cost of 20 to 40 units associated with extra JUMP instructions and stack operations required for separate function calls, unless the calling function becomes too complex.

### Inline modifiers used only once
In Solidity, modifiers provide a way to add reusable conditions or logic to functions. However, if a modifier is used only once, the modularization may not be beneficial in terms of gas efficiency. The overhead associated with a modifier – including two extra JUMP instructions and additional stack operations for the function call – results in an extra cost of about 20 to 40 gas. In cases where a modifier is unique to a single function and the function’s complexity does not necessitate breaking out logic, inlining the modifier's code directly within the function can be more gas-efficient. This approach avoids the overhead of a separate modifier call while maintaining code clarity.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

25:  modifier onlyAllowedCaller() {	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L25-L25), 


#### Recommendation

Evaluate the use of modifiers in your Solidity contracts, particularly those applied to only one function. If a modifier is unique to a single function, consider inlining its logic within the function itself to save gas. This is especially advantageous for simpler functions where the additional clarity provided by a separate modifier does not outweigh the gas cost of its use. Ensure that the function remains clear and maintainable after inlining the modifier's logic, and document the rationale for inlining to maintain code readability and facilitate future maintenance.

### Optimizing Arithmetic with Shift Operators for Division and Multiplication by Powers of Two
In computational operations, especially within contexts where efficiency matters, certain arithmetic operations can be optimized. One such optimization is leveraging shift operators for division and multiplication by powers of two. This stems from the binary nature of numbers in computing, where shifting bits to the left (using `<<`) effectively multiplies a number by 2 for each position shifted, and shifting bits to the right (using `>>`) divides the number by 2 for each position shifted.

In Solidity, and many other programming languages, using shift operators can be more gas-efficient than traditional arithmetic operations for these specific cases. For instance, instead of performing `value * 4`, one can use `value << 2`, and instead of `value / 4`, one can use `value >> 2`. These optimizations can result in reduced gas costs and faster execution times.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

173:    uint256 newEntropy = (forgerEntropy + mergerEntropy) / 2;	// @audit-issue
```
[173](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L173-L173), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

22:  uint256 public nukeFactorMaxParam = MAX_DENOMINATOR / 2;	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L22-L22), 


#### Recommendation

When performing division or multiplication by powers of two in Solidity, consider using shift operators (`<<` for multiplication and `>>` for division) for improved gas efficiency. This optimization takes advantage of the binary nature of computing and can lead to reduced gas costs and faster execution times.

### Redundant State Variable Getters in Solidity
In Solidity, state variables can have different visibility levels, including `public`. When a state variable is declared as `public`, the Solidity compiler automatically generates a getter function for it. This implicit getter has the same name as the state variable and allows external callers to query the variable's value.

A common oversight is the explicit creation of a function that returns the value of a public state variable. This function essentially duplicates the functionality already provided by the automatically generated getter. For instance, if there's a public state variable `uint256 public value;`, there's no need for a function like `function getValue() public view returns (uint256) { return value; }`, as the compiler already provides a `value()` function.



```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

130:  function getGeneration() public view returns (uint256) {	// @audit-issue
131:    return currentGeneration;
132:  }
```
[130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L130-L132), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

113:  function getAgeMultiplier() public view returns (uint256) {	// @audit-issue
114:    return ageMultiplier;
115:  }
```
[113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L113-L115), 


#### Recommendation

Avoid creating explicit getter functions for 'public' state variables in Solidity. The compiler automatically generates getters for such variables, making additional functions redundant. This practice helps reduce contract size, lowers deployment costs, and simplifies maintenance and understanding of the contract.

### Use `private` Rather than `public` for Constants 
In Solidity, constants represent immutable values that cannot be changed after they are set at compile-time. By default, constants have internal visibility, meaning they can be accessed within the contract they are declared in and in derived contracts. If a constant is explicitly declared as `public`, Solidity automatically generates a getter function for it. While this might seem harmless, it actually incurs a gas overhead, especially when the contract is deployed, as the EVM needs to generate bytecode for that getter. Conversely, declaring constants as `private` ensures that no additional getter is generated, optimizing gas usage.

```solidity
Path: ./contracts/NukeFund/NukeFund.sol

12:  uint256 public constant MAX_DENOMINATOR = 100000;	// @audit-issue
```
[12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L12-L12), 


#### Recommendation

To optimize gas usage in your Solidity contracts, declare constants with `private` visibility rather than `public` when possible. Using `private` prevents the automatic generation of a getter function, reducing gas overhead, especially during contract deployment.

### Consider pre-calculating the address of `address(this)` to save gas
Use `foundry`'s [`script.sol`](https://book.getfoundry.sh/reference/forge-std/compute-create-address) or `solady`'s [`LibRlp.sol`](https://github.com/Vectorized/solady/blob/main/src/utils/LibRLP.sol) to save the value in a constant, which will avoid having to spend gas to push the value on the stack every time it's used.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

359:    require(address(this).balance >= totalAmount, 'Insufficient balance');	// @audit-issue
```
[359](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L359-L359), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

87:    uint256 _rewardBalance = payable(address(this)).balance;	// @audit-issue
```
[87](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L87-L87), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

48:      nftContract.getApproved(tokenId) == address(this) ||	// @audit-issue

49:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue

53:    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract	// @audit-issue

81:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer NFT to the buyer	// @audit-issue

104:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer the nft back to seller	// @audit-issue

116:    emit NukeFundContribution(address(this), amount);	// @audit-issue
```
[48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L48-L48), [49](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L49-L49), [53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L53-L53), [81](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L81-L81), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L104-L104), [116](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L116-L116), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

159:      nftContract.getApproved(tokenId) == address(this) ||	// @audit-issue

160:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue
```
[159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L159-L159), [160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L160-L160), 


#### Recommendation

To enhance gas efficiency, cache the contract's address by storing `address(this)` in a state variable at the point of contract deployment or initialization. Use this cached address throughout the contract instead of repeatedly calling `address(this)`. This practice reduces the gas cost associated with multiple computations of the contract's address, leading to more efficient contract execution, especially in scenarios with frequent usage of the contract's address.

### All variables can be packed into fewer storage slots by truncating timestamp bytes
By using a `uint32` rather than a larger type for variables that track timestamps, one can save gas by using fewer storage slots per struct, at the expense of the protocol breaking after the year 2106 (when `uint32` wraps). If this is an acceptable tradeoff, if variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (**20000 gas**). Reads of the variables can also be cheaper

```solidity
Path: ./contracts/DevFund/DevFund.sol

64:    uint256 pending = info.pendingRewards +	// @audit-issue
```
[64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L64-L64), 


#### Recommendation

Evaluate the use of `uint32` for timestamp variables in your Solidity contracts, especially within structs that are frequently written to or read from storage. Ensure that the reduced size will not impact the functionality of your contract, particularly considering the 2106 overflow limitation. When implementing this optimization, aim to group such variables together in the same storage slot to maximize gas savings. This strategy is most effective when these variables are updated together, as in a constructor or specific functions, allowing for a single storage operation (`Gsset`) instead of multiple. Always balance the benefits of gas savings against the longevity and future-proofing of your contract.

### Counting down in for statements is more gas efficient
Looping downwards in Solidity is more gas efficient due to how the EVM compares variables. In a 'for' loop that counts down, the end condition is usually to compare with zero, which is cheaper than comparing with another number. As such, restructure your loops to count downwards where possible.

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

50:    for (uint256 i = 1; i <= listingCount; ++i) {	// @audit-issue
```
[50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L50-L50), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

52:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue

72:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue

90:      for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {	// @audit-issue
```
[52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L52-L52), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L72-L72), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L90-L90), 


#### Recommendation

Where feasible, refactor `for` loops in your Solidity contracts to count downwards. Adjust the loop initialization, condition, and iteration statements to decrement the loop variable and terminate the loop when it reaches zero. This approach can lead to gas savings, making your contract more efficient in terms of execution costs. Ensure that this refactoring aligns with the logic and requirements of your contract, and thoroughly test to confirm that the revised loop behavior matches the intended functionality.

### Use solady library where possible to save gas
The following OpenZeppelin imports have a Solady equivalent, as such they can be used to save GAS as Solady modules have been specifically designed to be as GAS efficient as possible

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

7:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L7-L7), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

4:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

5:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L6-L6), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L6-L6), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

7:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L7-L7), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L6-L6), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L5-L5), 


#### Recommendation

Evaluate and, where appropriate, integrate Solady modules in your Solidity contracts as alternatives to similar OpenZeppelin imports. Focus on areas where gas efficiency can be significantly improved. Ensure that any replacement with Solady's modules does not compromise the security or functionality of your contracts. Conduct thorough testing and code reviews when making such substitutions to confirm compatibility and maintain the integrity of your application. Stay informed about updates and community feedback on both libraries to make informed decisions about their use in your projects.

### Consider using solady's 'FixedPointMathLib'
Using Solady's "FixedPointMathLib" for multiplication or division operations in Solidity can lead to significant gas savings. This library is designed to optimize fixed-point arithmetic operations, which are common in financial calculations involving tokens or currencies. By implementing more efficient algorithms and assembly optimizations, "FixedPointMathLib" minimizes the computational resources required for these operations. For contracts that frequently perform such calculations, integrating this library can reduce transaction costs, thereby enhancing overall performance and cost-effectiveness. However, developers must ensure compatibility with their existing codebase and thoroughly test for accuracy and expected behavior to avoid any unintended consequences.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

173:    uint256 newEntropy = (forgerEntropy + mergerEntropy) / 2;	// @audit-issue

229:    uint256 priceIncrease = priceIncrement * currentGenMintCount;	// @audit-issue
```
[173](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L173-L173), [229](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L229-L229), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

86:    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue

138:    uint8 mergerForgePotential = uint8((mergerEntropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue

146:    uint256 devFee = forgingFee / taxCut;	// @audit-issue
```
[86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L86-L86), [138](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L138-L138), [146](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L146-L146), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

16:      uint256 amountPerWeight = msg.value / totalDevWeight;	// @audit-issue

17:      uint256 remaining = msg.value - (amountPerWeight * totalDevWeight);	// @audit-issue

45:    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;	// @audit-issue

55:    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;	// @audit-issue

65:      (totalRewardDebt - info.rewardDebt) *	// @audit-issue

80:      info.pendingRewards + (totalRewardDebt - info.rewardDebt) * info.weight;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L16-L16), [17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L17-L17), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L45-L45), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L55-L55), [65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L65-L65), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L80-L80), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

72:    uint256 nukeFundContribution = msg.value / taxCut;	// @audit-issue
```
[72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L72-L72), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

41:    uint256 devShare = msg.value / taxCut; // Calculate developer's share (10%)	// @audit-issue

121:    uint256 daysOld = (block.timestamp -	// @audit-issue

128:    uint256 age = (daysOld *	// @audit-issue

145:    uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits	// @audit-issue

147:    uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) /	// @audit-issue

166:    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; // Calculate the potential claim amount based on the finalNukeFactor	// @audit-issue

167:    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size	// @audit-issue
```
[41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L41-L41), [121](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L121-L121), [128](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L128-L128), [145](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L145-L145), [147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L147-L147), [166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L166-L166), [167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L167-L167), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

152:    nukeFactor = entropy / 4000000;	// @audit-issue

177:    uint256 position = numberIndex * 6; // calculate the position for slicing the entropy value	// @audit-issue

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue

182:    uint256 paddedEntropy = entropy * (10 ** (6 - numberOfDigits(entropy)));	// @audit-issue

191:      number /= 10;	// @audit-issue

200:      number /= 10;	// @audit-issue
```
[152](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L152-L152), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L177-L177), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), [182](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L182-L182), [191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L191-L191), [200](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L200-L200), 


#### Recommendation

Consider integrating Solady's 'FixedPointMathLib' into your Solidity contracts for optimized fixed-point arithmetic operations. This library can provide substantial gas savings and enhance the performance of your contract. Before integration, evaluate how 'FixedPointMathLib' aligns with your contract’s requirements. Ensure thorough testing for accuracy and compatibility with your existing contract logic. Carefully document any changes and keep track of how these optimizations affect your contract's operations to maintain transparency and reliability in your application. Adopting 'FixedPointMathLib' should be a considered decision, balancing the benefits of gas efficiency with the need for maintaining code clarity and functionality.

### Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Each operation involving a `uint8` costs an extra [22-28](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

22:  mapping(uint256 => uint8) public forgingCounts; // track forgePotential	// @audit-issue

86:    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue

138:    uint8 mergerForgePotential = uint8((mergerEntropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L22-L22), [86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L86-L86), [138](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L138-L138), 


#### Recommendation

Minimize gas overhead by using 'uint256' or 'int256' instead of smaller integer types in Solidity contracts. The EVM operates more efficiently with 32-byte sizes. Downcast to smaller types only when necessary, as operations with smaller types like 'uint8' incur extra gas due to additional EVM operations for size adjustment.

### Refactor modifiers to call a local function
Modifiers code is copied in all instances where it's used, increasing bytecode size. If deployment gas costs are a concern for this contract, refactoring modifiers into functions can reduce bytecode size significantly at the cost of one JUMP.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {	// @audit-issue
```
[51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L51-L51), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

25:  modifier onlyAllowedCaller() {	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L25-L25), 


#### Recommendation

Evaluate your contract's use of modifiers, particularly those applied across multiple functions, to identify candidates for refactoring into functions. Convert these modifiers into external or public functions that perform the same checks or actions. Then, replace the modifier usage in function declarations with calls to these functions at the start of your functions

### Avoid Unnecessary Public Variables
Public state variables in Solidity automatically generate getter functions, increasing contract size and potentially leading to higher deployment and interaction costs. To optimize gas usage and contract efficiency, minimize the use of public variables unless external access is necessary. Instead, use internal or private visibility combined with explicit getter functions when required. This practice not only reduces contract size but also provides better control over data access and manipulation, enhancing security and readability. Prioritize lean, efficient contracts to ensure cost-effectiveness and better performance on the blockchain.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

22:  uint256 public maxTokensPerGen = 10000;	// @audit-issue

23:  uint256 public startPrice = 0.005 ether;	// @audit-issue

24:  uint256 public priceIncrement = 0.0000245 ether;	// @audit-issue

25:  uint256 public priceIncrementByGen = 0.000005 ether;	// @audit-issue

27:  IEntityForging public entityForgingContract;	// @audit-issue

28:  IEntropyGenerator public entropyGenerator;	// @audit-issue

29:  IAirdrop public airdropContract;	// @audit-issue

30:  address public nukeFundAddress;	// @audit-issue

33:  uint256 public currentGeneration = 1;	// @audit-issue

35:  uint256 public maxGeneration = 10;	// @audit-issue

37:  bytes32 public rootHash;	// @audit-issue

39:  uint256 public whitelistEndTime;	// @audit-issue

42:  mapping(uint256 => uint256) public tokenCreationTimestamps;	// @audit-issue

43:  mapping(uint256 => uint256) public lastTokenTransferredTimestamp;	// @audit-issue

44:  mapping(uint256 => uint256) public tokenEntropy;	// @audit-issue

45:  mapping(uint256 => uint256) public generationMintCounts;	// @audit-issue

46:  mapping(uint256 => uint256) public tokenGenerations;	// @audit-issue

47:  mapping(uint256 => address) public initialOwners;	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L22-L22), [23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L23-L23), [24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L24-L24), [25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L25-L25), [27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L27-L27), [28](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L28-L28), [29](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L29-L29), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L30-L30), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L33-L33), [35](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L35-L35), [37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L37-L37), [39](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L39-L39), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L43-L43), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L44-L44), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L45-L45), [46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L46-L46), [47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L47-L47), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

11:  ITraitForgeNft public nftContract;	// @audit-issue

12:  address payable public nukeFundAddress;	// @audit-issue

13:  uint256 public taxCut = 10;	// @audit-issue

14:  uint256 public oneYearInDays = 365 days;	// @audit-issue

15:  uint256 public listingCount = 0;	// @audit-issue

16:  uint256 public minimumListFee = 0.01 ether;	// @audit-issue

19:  mapping(uint256 => uint256) public listedTokenIds;	// @audit-issue

21:  mapping(uint256 => Listing) public listings;	// @audit-issue

22:  mapping(uint256 => uint8) public forgingCounts; // track forgePotential	// @audit-issue
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L12-L12), [13](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L13-L13), [14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L14-L14), [15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L15-L15), [16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L16-L16), [19](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L19-L19), [21](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L21-L21), [22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L22-L22), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

10:  uint256 public totalDevWeight;	// @audit-issue

11:  uint256 public totalRewardDebt;	// @audit-issue

12:  mapping(address => DevInfo) public devInfo;	// @audit-issue
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L10-L10), [11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L12-L12), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

12:  ITraitForgeNft public nftContract;	// @audit-issue

13:  address payable public nukeFundAddress;	// @audit-issue

14:  uint256 public taxCut = 10;	// @audit-issue

16:  uint256 public listingCount = 0;	// @audit-issue

18:  mapping(uint256 => uint256) public listedTokenIds;	// @audit-issue

20:  mapping(uint256 => Listing) public listings;	// @audit-issue
```
[12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L12-L12), [13](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L13-L13), [14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L14-L14), [16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L16-L16), [18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L18-L18), [20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L20-L20), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

12:  uint256 public constant MAX_DENOMINATOR = 100000;	// @audit-issue

15:  ITraitForgeNft public nftContract;	// @audit-issue

16:  IAirdrop public airdropContract;	// @audit-issue

17:  address payable public devAddress;	// @audit-issue

18:  address payable public daoAddress;	// @audit-issue

19:  uint256 public taxCut = 10;	// @audit-issue

20:  uint256 public defaultNukeFactorIncrease = 250;	// @audit-issue

21:  uint256 public maxAllowedClaimDivisor = 2;	// @audit-issue

22:  uint256 public nukeFactorMaxParam = MAX_DENOMINATOR / 2;	// @audit-issue

23:  uint256 public minimumDaysHeld = 3 days;	// @audit-issue

24:  uint256 public ageMultiplier;	// @audit-issue
```
[12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L12-L12), [15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L15-L15), [16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L16-L16), [17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L17-L17), [18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L18-L18), [19](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L19-L19), [20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L20-L20), [21](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L21-L21), [22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L22-L22), [23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L23-L23), [24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L24-L24), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

19:  uint256 public slotIndexSelectionPoint;	// @audit-issue

20:  uint256 public numberIndexSelectionPoint;	// @audit-issue
```
[19](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L19-L19), [20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L20-L20), 


#### Recommendation

Avoid creating explicit getter functions for 'public' state variables in Solidity. The compiler automatically generates getters for such variables, making additional functions redundant. This practice helps reduce contract size, lowers deployment costs, and simplifies maintenance and understanding of the contract.

### Use `do while` loops intead of for loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration.
```solidity
uint256 i = 1;
do {                   
    param2 += i;
    i++;
}
while (i < 50);
``` 
is better than
```solidity
for(uint256 i = 1; i< 50; i++){
    param1 += i;
}
```


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

50:    for (uint256 i = 1; i <= listingCount; ++i) {	// @audit-issue
```
[50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L50-L50), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

52:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue

72:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue

90:      for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {	// @audit-issue
```
[52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L52-L52), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L72-L72), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L90-L90), 


#### Recommendation

Where appropriate, consider using a `do while` loop instead of a `for` loop in your Solidity contracts. This is especially beneficial when the first iteration of the loop does not require a condition check. Refactor your loop logic to fit the `do while` structure for more gas-efficient execution. However, ensure that the loop's logic and termination conditions are correctly implemented to avoid infinite loops or other logical errors. Always balance gas efficiency with code readability and the specific requirements of your contract's logic.

### Using XOR (^) and AND (&) bitwise equivalents for gas optimizations
Given 4 variables a, b, c and d represented as such:
```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d
```
To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

160:      msg.sender == address(entityForgingContract),	// @audit-issue

277:    return roleIndicator == 0;	// @audit-issue

332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration	// @audit-issue

386:        listing.tokenId == firstTokenId &&	// @audit-issue

387:        listing.account == from &&	// @audit-issue
```
[160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L160-L160), [277](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L277-L277), [332](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L332-L332), [386](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L386-L386), [387](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L387-L387), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

75:      nftContract.ownerOf(tokenId) == msg.sender,	// @audit-issue

92:    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy	// @audit-issue

112:      nftContract.ownerOf(mergerTokenId) == msg.sender,	// @audit-issue

120:      nftContract.getTokenGeneration(mergerTokenId) ==	// @audit-issue

181:      nftContract.ownerOf(tokenId) == msg.sender ||	// @audit-issue

182:        msg.sender == address(nftContract),	// @audit-issue

201:    if (lastForgeResetTimestamp[tokenId] == 0) {	// @audit-issue
```
[75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L75-L75), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L92-L92), [112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L112-L112), [120](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L120-L120), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L181-L181), [182](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L182-L182), [201](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L201-L201), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

33:    require(info.weight == 0, 'Already registered');	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L33-L33), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

44:      nftContract.ownerOf(tokenId) == msg.sender,	// @audit-issue

48:      nftContract.getApproved(tokenId) == address(this) ||	// @audit-issue

66:      msg.value == listing.price,	// @audit-issue

99:      listing.seller == msg.sender,	// @audit-issue
```
[44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L44-L44), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L48-L48), [66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L66-L66), [99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L99-L99), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

159:      nftContract.getApproved(tokenId) == address(this) ||	// @audit-issue
```
[159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L159-L159), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

26:    require(msg.sender == allowedCaller, 'Caller is not allowed');	// @audit-issue

158:    isForger = role == 0;	// @audit-issue

171:      slotIndex == slotIndexSelectionPoint &&	// @audit-issue

172:      numberIndex == numberIndexSelectionPoint	// @audit-issue
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L26-L26), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L158-L158), [171](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L171-L171), [172](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L172-L172), 


#### Recommendation

Review your Solidity contracts to identify any computations that are performed multiple times with the same inputs. Cache the results of these computations in local variables and reuse them within the function or across function calls if the state remains unchanged.

### The result of a function call should be cached rather than re-calling the function
The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.        

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

219:      mintPrice = calculateMintPrice();	// @audit-issue: Function call `calculateMintPrice` is called multiple times at lines [211].
```
[219](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L219-L219), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

116:      nftContract.ownerOf(forgerTokenId) != msg.sender,	// @audit-issue: Function call `ownerOf` is called multiple times at lines [148].
```
[116](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L116-L116), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

24:      (bool success, ) = payable(owner()).call{ value: msg.value }('');	// @audit-issue: Function call `payable` is called multiple times at lines [20].

20:        (bool success, ) = payable(owner()).call{ value: remaining }('');	// @audit-issue: Function call `owner` is called multiple times at lines [24].

25:      require(success, 'Failed to send Ether to owner');	// @audit-issue: Function call `require` is called multiple times at lines [21].
```
[24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L24-L24), [20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L20-L20), [25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L25-L25), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

55:      require(success, 'ETH send failed');	// @audit-issue: Function call `require` is called multiple times at lines [52, 48].

49:      emit DevShareDistributed(devShare);	// @audit-issue: Function call `DevShareDistributed` is called multiple times at lines [56].
```
[55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L55-L55), [49](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L49-L49), 


#### Recommendation

Cache the result of function calls in Solidity instead of making repeated calls to the same function. This practice significantly reduces gas consumption by minimizing costly function call operations.

### Avoid updating storage when the value hasn't changed
Manipulating storage in solidity is gas-intensive. It can be optimized by avoiding unnecessary storage updates when the new value equals the existing value. If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas).

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

69:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

101:    startPrice = _startPrice;	// @audit-issue

105:    priceIncrement = _priceIncrement;	// @audit-issue

111:    priceIncrementByGen = _priceIncrementByGen;	// @audit-issue

123:    rootHash = rootHash_;	// @audit-issue

127:    whitelistEndTime = endTime_;	// @audit-issue

294:    initialOwners[newItemId] = to;	// @audit-issue

326:    tokenEntropy[newTokenId] = entropy;	// @audit-issue

327:    tokenGenerations[newTokenId] = gen;	// @audit-issue

329:    initialOwners[newTokenId] = newOwner;	// @audit-issue
```
[69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L69-L69), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L101-L101), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L105-L105), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L111-L111), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L123-L123), [127](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L127-L127), [294](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L294-L294), [326](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L326-L326), [327](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L327-L327), [329](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L329-L329), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

33:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

37:    taxCut = _taxCut;	// @audit-issue

41:    oneYearInDays = value;	// @audit-issue

45:    minimumListFee = _fee;	// @audit-issue

96:    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L33-L33), [37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L37-L37), [41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L41-L41), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L45-L45), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L96-L96), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

30:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

34:    taxCut = _taxCut;	// @audit-issue

56:    listings[listingCount] = Listing(msg.sender, tokenId, price, true);	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L30-L30), [34](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L34-L34), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L56-L56), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

64:    taxCut = _taxCut;	// @audit-issue

68:    minimumDaysHeld = value;	// @audit-issue

72:    defaultNukeFactorIncrease = value;	// @audit-issue

76:    maxAllowedClaimDivisor = value;	// @audit-issue

80:    nukeFactorMaxParam = value;	// @audit-issue

85:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

90:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

95:    devAddress = account;	// @audit-issue

100:    daoAddress = account;	// @audit-issue

110:    ageMultiplier = _ageMultiplier;	// @audit-issue
```
[64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L64-L64), [68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L68-L68), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L72-L72), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L76-L76), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L80-L80), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L85-L85), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L90-L90), [95](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L95-L95), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L100-L100), [110](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L110-L110), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

37:    allowedCaller = _allowedCaller;	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L37-L37), 


#### Recommendation

Optimize gas usage by avoiding storage updates in Solidity when the new value is the same as the existing one. This prevents unnecessary gas expenditure from storage resets, balancing the cost against cold or warm storage access as needed.

### Functions guaranteed to `revert` when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

66:  function setNukeFundContract(	// @audit-issue

74:  function setEntityForgingContract(	// @audit-issue

82:  function setEntropyGenerator(	// @audit-issue

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue

96:  function startAirdrop(uint256 amount) external whenNotPaused onlyOwner {	// @audit-issue

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {	// @audit-issue

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {	// @audit-issue

108:  function setPriceIncrementByGen(	// @audit-issue

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue
```
[66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L66-L66), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L74-L74), [82](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L82-L82), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L96-L96), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L100), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L104), [108](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L108), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

30:  function setNukeFundAddress(	// @audit-issue

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue

40:  function setOneYearInDays(uint256 value) external onlyOwner {	// @audit-issue

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L36), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L44), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

30:  function addDev(address user, uint256 weight) external onlyOwner {	// @audit-issue

40:  function updateDev(address user, uint256 weight) external onlyOwner {	// @audit-issue

51:  function removeDev(address user) external onlyOwner {	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L30-L30), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L40-L40), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L51-L51), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

27:  function setNukeFundAddress(	// @audit-issue

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L27), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L33), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

63:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue

67:  function setMinimumDaysHeld(uint256 value) external onlyOwner {	// @audit-issue

71:  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {	// @audit-issue

75:  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {	// @audit-issue

79:  function setNukeFactorMaxParam(uint256 value) external onlyOwner {	// @audit-issue

84:  function setTraitForgeNftContract(address _traitForgeNft) external onlyOwner {	// @audit-issue

89:  function setAirdropContract(address _airdrop) external onlyOwner {	// @audit-issue

94:  function setDevAddress(address payable account) external onlyOwner {	// @audit-issue

99:  function setDaoAddress(address payable account) external onlyOwner {	// @audit-issue

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {	// @audit-issue
```
[63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L63), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L67-L67), [71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L71-L71), [75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L75-L75), [79](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L79-L79), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L84-L84), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L89-L89), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L94-L94), [99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L99-L99), [109](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L109), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

36:  function setAllowedCaller(address _allowedCaller) external onlyOwner {	// @audit-issue

101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {	// @audit-issue

206:  function initializeAlphaIndices() public whenNotPaused onlyOwner {	// @audit-issue
```
[36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L36-L36), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L101-L101), [206](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L206-L206), 


#### Recommendation

Mark functions with access restrictions like 'onlyOwner' as 'payable' in Solidity. This reduces gas costs for legitimate callers by removing the compiler's checks for incoming payments, as the function will revert for unauthorized users anyway.

### Multiple mappings can be replaced with a single struct mapping
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

46:  mapping(uint256 => uint256) public tokenGenerations;	// @audit-issue: Can be combined with: 
	 tokenCreationTimestamps in TraitForgeNft at line: 42
	 lastTokenTransferredTimestamp in TraitForgeNft at line: 43
	 tokenEntropy in TraitForgeNft at line: 44
	 generationMintCounts in TraitForgeNft at line: 45
```
[46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L46-L46), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

23:  mapping(uint256 => uint256) private lastForgeResetTimestamp;	// @audit-issue: Can be combined with: 
	 listedTokenIds in EntityForging at line: 19
```
[23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L23-L23), 


#### Recommendation

Refactor your Solidity contracts to replace multiple related mappings with a single mapping that uses a struct. This struct should contain all the fields that were previously in separate mappings. This consolidation optimizes storage usage by reducing the number of storage slots and hash computations required, leading to significant gas savings. Ensure that the fields within the struct are logically related and frequently accessed or modified together. This strategy is most effective when the size of struct fields allows them to fit within a single storage slot, maximizing the benefits of the EVM's storage packing capabilities.

### `x += y` costs more gas than `x = x + y` for stack variables
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

218:      budgetLeft -= mintPrice;	// @audit-issue
```
[218](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L218-L218), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

45:    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;	// @audit-issue

55:    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;	// @audit-issue
```
[45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L45-L45), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L55-L55), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

191:      number /= 10;	// @audit-issue

200:      number /= 10;	// @audit-issue
```
[191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L191-L191), [200](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L200-L200), 


#### Recommendation

Prefer using 'x = x + y' over 'x += y' for state variable assignments in Solidity to save gas. The latter incurs extra costs due to additional JUMP instructions and stack operations associated with non-inlined function calls.

### Use calldata instead of memory for function arguments that do not get mutated
Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

153:  function forge(
154:    address newOwner,
155:    uint256 parent1Id,
156:    uint256 parent2Id,
157:    string memory	// @audit-issue
158:  ) external whenNotPaused nonReentrant returns (uint256) {
```
[157](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L158), 


#### Recommendation

To optimize gas usage in your Solidity functions, mark data types as `calldata` instead of `memory` wherever applicable. This prevents unnecessary data loading into memory. Use `calldata` for function arguments that do not require changes within the function, except when passing them into another function that explicitly requires `memory` storage.

### Splitting `require()` statements that use `&&` saves gas
Instead of using the `&&` operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save 3 GAS per `&&`:

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

87:    require(	// @audit-issue
88:      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
89:      'Entity has reached its forging limit'
90:    );

140:    require(	// @audit-issue
141:      mergerForgePotential > 0 &&
142:        forgingCounts[mergerTokenId] <= mergerForgePotential,
143:      'forgePotential insufficient'
144:    );
```
[87](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L87-L90), [140](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L140-L144), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

65:    require(	// @audit-issue
66:      lastInitializedIndex >= batchSize1 && lastInitializedIndex < batchSize2,
67:      'Batch 2 not ready or already initialized.'
68:    );

85:    require(	// @audit-issue
86:      lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,
87:      'Batch 3 not ready or already completed.'
88:    );
```
[65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L65-L68), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L85-L88), 


#### Recommendation

Split 'require()' statements in Solidity that use '&&' into multiple 'require()' statements, each with a single condition. This approach saves 3 gas per '&&', optimizing overall gas usage in the contract.

### Nesting `if` statements that uses `&&` saves gas
In Solidity, the way conditional checks are structured can impact the gas consumption of a transaction. When conditions are combined using `&&` within an `if` statement, Solidity short-circuits the evaluation, meaning that if the first condition is `false`, the subsequent conditions won't be evaluated. This behavior can lead to gas savings compared to using separate nested `if` statements because not all conditions might need to be checked every time. By efficiently structuring these conditional checks, contracts can optimize the gas required for execution, leading to reduced costs for users.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration	// @audit-issue

386:        listing.tokenId == firstTokenId &&	// @audit-issue
387:        listing.account == from &&
388:        listing.isListed
```
[332](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L332-L332), [386](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L386-L388), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

171:      slotIndex == slotIndexSelectionPoint &&	// @audit-issue
172:      numberIndex == numberIndexSelectionPoint
```
[171](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L171-L172), 


#### Recommendation


When multiple conditions need to be checked successively, try to combine them in a single `if` statement using `&&` instead of nesting separate `if` statements. This will leverage short-circuit evaluation for potential gas savings.


### Use `!= 0` Instead of `> 0` for Unsigned Integer Comparison
In Solidity, unsigned integers (e.g., `uint256`, `uint8`, etc.) represent non-negative whole numbers, ranging from 0 to a maximum value determined by their bit size. When performing comparisons on these numbers, especially to check if they are non-zero, developers have options. A common practice is to compare against zero using the `>` operator, as in `value > 0`. However, given the nature of unsigned integers, a more straightforward and slightly gas-efficient comparison is to use the `!=` operator, as in `value != 0`.

The primary rationale is that the `!=` comparison directly checks for non-equality, whereas the `>` comparison checks if one value is strictly greater than another. For unsigned integers, where negative values don't exist, the `!= 0` check is both semantically clearer and potentially more efficient at the EVM bytecode level.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

196:    if (excessPayment > 0) {	// @audit-issue

221:    if (budgetLeft > 0) {	// @audit-issue

381:    if (listedId > 0) {	// @audit-issue
```
[196](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L196-L196), [221](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L221-L221), [381](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L381-L381), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

88:      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,	// @audit-issue

141:      mergerForgePotential > 0 &&	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L88-L88), [141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L141-L141), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

15:    if (totalDevWeight > 0) {	// @audit-issue

19:      if (remaining > 0) {	// @audit-issue

32:    require(weight > 0, 'Invalid weight');	// @audit-issue

42:    require(weight > 0, 'Invalid weight');	// @audit-issue

53:    require(info.weight > 0, 'Not dev address');	// @audit-issue

68:    if (pending > 0) {	// @audit-issue
```
[15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L15-L15), [19](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L19-L19), [32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L32-L32), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L42-L42), [53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L53-L53), [68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L68-L68), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

42:    require(price > 0, 'Price must be greater than zero');	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L42-L42), 


#### Recommendation

Use '!= 0' instead of '> 0' for comparing unsigned integers in Solidity. This is semantically clearer and slightly more gas-efficient, as it directly checks for non-zero values without considering unnecessary relational comparisons.

### Constructor Can Be Marked As Payable
`payable` functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided.

A `constructor` can safely be marked as `payable`, since only the deployer would be able to pass funds, and the project itself would not pass any funds.



```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

61:  constructor() ERC721('TraitForgeNft', 'TFGNFT') {	// @audit-issue
```
[61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L61-L61), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

25:  constructor(address _traitForgeNft) {	// @audit-issue
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L25-L25), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

22:  constructor(address _traitForgeNft) {	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L22-L22), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

27:  constructor(	// @audit-issue
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L27-L27), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

30:  constructor(address _traitForgetNft) {	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L30-L30), 


#### Recommendation

Mark constructors as 'payable' in Solidity contracts to reduce gas costs, as this eliminates the need for the compiler to add checks against incoming payments. This is safe because only the deployer can send funds during contract creation, and typically no funds are sent at this stage.

### Use `selfbalance()` instead of `address(this).balance`
In Solidity, contracts often need to query their own Ether balance. While `address(this).balance` has been a widely used approach to retrieve the current contract's balance, it is not the most gas-efficient method, especially with the introduction of the `SELFBALANCE` opcode in more recent EVM versions.

The `SELFBALANCE` opcode provides a more gas-efficient way to obtain the balance of the current contract. In Solidity, this can be invoked using the `selfbalance()` function. Compared to the `BALANCE` opcode used by `address(this).balance`, `SELFBALANCE` offers significant gas savings:

- `BALANCE`: The static gas cost is 0. If the accessed address is warm, the dynamic gas cost is 100. If the address is cold, the dynamic cost is 2,600.
  
- `SELFBALANCE`: Semantically equivalent to the `BALANCE` opcode when called on the contract's own address, but with a dramatically reduced minimum gas cost of 5.

Switching to `selfbalance()` can lead to significant gas savings, especially in operations or functions that frequently check the contract's balance.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

359:    require(address(this).balance >= totalAmount, 'Insufficient balance');	// @audit-issue
```
[359](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L359-L359), 


#### Recommendation

To optimize gas usage when querying a contract's own Ether balance in Solidity, it's recommended to use the `selfbalance()` function, which utilizes the more gas-efficient `SELFBALANCE` opcode. This can result in significant gas savings compared to `address(this).balance`.

### Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L14), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

10:contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {	// @audit-issue
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L10-L10), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

9:contract DevFund is IDevFund, Ownable, ReentrancyGuard, Pausable {	// @audit-issue
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

11:contract EntityTrading is IEntityTrading, ReentrancyGuard, Ownable, Pausable {	// @audit-issue
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L11-L11), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

11:contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {	// @audit-issue
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L11-L11), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

9:contract EntropyGenerator is IEntropyGenerator, Ownable, Pausable {	// @audit-issue
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L9-L9), 


#### Recommendation

Optimize gas usage by renaming 'public'/'external' functions and 'public' member variables in Solidity. Aim for shorter and more efficient names, especially for frequently called functions. This can save gas during deployment and reduce gas costs per call due to lower method ID sorting positions.

### Consider activating `via-ir` for deploying
The IR-based code generator was developed to make code generation more performant by enabling optimization passes that can be applied across functions.

It is possible to activate the IR-based code generator through the command line by using the flag `--via-ir`or by including the option `{"viaIR": true}`.

Keep in mind that compiling with this option may take longer. However, you can simply test it before deploying your code. If you find that it provides better performance, you can add the `--via-ir` flag to your deploy command.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

Consider activating `via-ir`.

### Optimize Deployment Size by Fine-tuning IPFS Hash
The Solidity compiler appends 53 bytes of metadata to the smart contract code, incurring an extra cost of 10,600 gas. This additional expense arises from 200 gas per bytecode, plus calldata cost, which amounts to 16 gas for non-zero bytes and 4 gas for zero bytes. This results in a maximum of 848 extra gas in calldata cost.

Reducing this cost is crucial for the following reasons:

The metadata's 53-byte addition leads to a deployment cost increase of 10,600 gas. It can also result in an additional calldata cost of up to 848 gas. Ways to Minimize Gas Consumption:

Employ the `--no-cbor-metadata` compiler option to exclude metadata. Be cautious as this might impact contract verification. Search for code comments that yield an IPFS hash with more zeros, thereby reducing calldata costs.

```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

To optimize deployment size and reduce associated costs, consider the following strategies:
1. **Exclude Metadata with Compiler Option**: Use the `solc` compiler’s `--metadata-hash none` or `--no-cbor-metadata` option to prevent the inclusion of metadata in the compiled bytecode. This action reduces the bytecode size, thus lowering deployment gas costs. However, exercise caution with this approach, as it might affect the ability to verify the contract on platforms like Etherscan.

2. **Optimize IPFS Hash for More Zeros**: If excluding metadata is not desirable, another approach involves optimizing code comments or elements that influence the metadata hash generation to achieve an IPFS hash with a higher proportion of zeros. Since calldata costs are lower for zero bytes, a metadata hash with more zeros can reduce the calldata costs associated with contract interactions.

Example for excluding metadata:
```bash
solc --metadata-hash none YourContract.sol
```


### Low level `call` can be optimized with assembly
`returnData` is copied to memory even if the variable is not utilized: the proper way to handle this is through a low level assembly call.
```solidity
// before
(bool success,) = payable(receiver).call{gas: gas, value: value}("");

//after
bool success;
assembly {
    success := call(gas, receiver, value, 0, 0, 0, 0)
}
```


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

197:      (bool refundSuccess, ) = msg.sender.call{ value: excessPayment }('');	// @audit-issue

222:      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');	// @audit-issue

361:    (bool success, ) = nukeFundAddress.call{ value: totalAmount }('');	// @audit-issue
```
[197](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L197-L197), [222](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L222-L222), [361](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L361-L361), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

156:    (bool success, ) = nukeFundAddress.call{ value: devFee }('');	// @audit-issue

158:    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');	// @audit-issue
```
[156](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L156-L156), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L158-L158), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

20:        (bool success, ) = payable(owner()).call{ value: remaining }('');	// @audit-issue

24:      (bool success, ) = payable(owner()).call{ value: msg.value }('');	// @audit-issue

89:    (bool success, ) = payable(to).call{ value: amount }('');	// @audit-issue
```
[20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L20-L20), [24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L24-L24), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L89-L89), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

77:    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(	// @audit-issue

114:    (bool success, ) = nukeFundAddress.call{ value: amount }('');	// @audit-issue
```
[77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L77-L77), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L114-L114), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

47:      (bool success, ) = devAddress.call{ value: devShare }('');	// @audit-issue

51:      (bool success, ) = payable(owner()).call{ value: devShare }('');	// @audit-issue

54:      (bool success, ) = daoAddress.call{ value: devShare }('');	// @audit-issue

177:    (bool success, ) = payable(msg.sender).call{ value: claimAmount }('');	// @audit-issue
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L47), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L51-L51), [54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L54-L54), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L177-L177), 


#### Recommendation

Optimize low-level 'call' operations in Solidity using assembly, especially when 'returnData' is not needed. This avoids unnecessary copying to memory and can lead to gas savings. For example, replace '(bool success,) = payable(receiver).call{gas: gas, value: value}("");' with an assembly block: 'assembly { success := call(gas, receiver, value, 0, 0, 0, 0) }'. This ensures a more efficient execution.

### Assembly: Use scratch space for building calldata
If an external call's calldata can fit into two or fewer words, use the scratch space to build the calldata, rather than allowing Solidity to do a memory expansion.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

97:    airdropContract.startAirdrop(amount);	// @audit-issue

146:    if (!airdropContract.airdropStarted()) {	// @audit-issue

148:      airdropContract.subUserAmount(initialOwners[tokenId], entropy);	// @audit-issue

288:    uint256 entropyValue = entropyGenerator.getNextEntropy();	// @audit-issue

296:    if (!airdropContract.airdropStarted()) {	// @audit-issue

297:      airdropContract.addUserAmount(to, entropyValue);	// @audit-issue

337:    if (!airdropContract.airdropStarted()) {	// @audit-issue

338:      airdropContract.addUserAmount(newOwner, entropy);	// @audit-issue

353:    entropyGenerator.initializeAlphaIndices();	// @audit-issue

375:    uint listedId = entityForgingContract.getListedTokenIds(firstTokenId);	// @audit-issue

382:      IEntityForging.Listing memory listing = entityForgingContract.getListings(	// @audit-issue

390:        entityForgingContract.cancelListingForForging(firstTokenId);	// @audit-issue
```
[97](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L97-L97), [146](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L146-L146), [148](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L148-L148), [288](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L288-L288), [296](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L296-L296), [297](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L297-L297), [337](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L337-L337), [338](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L338-L338), [353](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L353-L353), [375](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L375-L375), [382](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L382-L382), [390](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L390-L390), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

75:      nftContract.ownerOf(tokenId) == msg.sender,	// @audit-issue

85:    uint256 entropy = nftContract.getTokenEntropy(tokenId); // Retrieve entropy for tokenId	// @audit-issue

112:      nftContract.ownerOf(mergerTokenId) == msg.sender,	// @audit-issue

116:      nftContract.ownerOf(forgerTokenId) != msg.sender,	// @audit-issue

120:      nftContract.getTokenGeneration(mergerTokenId) ==	// @audit-issue

121:        nftContract.getTokenGeneration(forgerTokenId),	// @audit-issue

136:    uint256 mergerEntropy = nftContract.getTokenEntropy(mergerTokenId);	// @audit-issue

148:    address payable forgerOwner = payable(nftContract.ownerOf(forgerTokenId));	// @audit-issue

164:    uint256 newEntropy = nftContract.getTokenEntropy(newTokenId);	// @audit-issue

181:      nftContract.ownerOf(tokenId) == msg.sender ||	// @audit-issue
```
[75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L75-L75), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L85-L85), [112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L112-L112), [116](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L116-L116), [120](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L120-L120), [121](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L121-L121), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L136-L136), [148](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L148-L148), [164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L164-L164), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L181-L181), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

44:      nftContract.ownerOf(tokenId) == msg.sender,	// @audit-issue

48:      nftContract.getApproved(tokenId) == address(this) ||	// @audit-issue

49:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue
```
[44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L44-L44), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L48-L48), [49](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L49-L49), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

46:    if (!airdropContract.airdropStarted()) {	// @audit-issue

50:    } else if (!airdropContract.daoFundAllowed()) {	// @audit-issue

119:    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');	// @audit-issue

122:      nftContract.getTokenCreationTimestamp(tokenId)) /	// @audit-issue

126:    uint256 perfomanceFactor = nftContract.getTokenEntropy(tokenId) % 10;	// @audit-issue

138:      nftContract.ownerOf(tokenId) != address(0),	// @audit-issue

142:    uint256 entropy = nftContract.getTokenEntropy(tokenId);	// @audit-issue

155:      nftContract.isApprovedOrOwner(msg.sender, tokenId),	// @audit-issue

159:      nftContract.getApproved(tokenId) == address(this) ||	// @audit-issue

160:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue

176:    nftContract.burn(tokenId); // Burn the token	// @audit-issue

187:      nftContract.ownerOf(tokenId) != address(0),	// @audit-issue

191:      nftContract.getTokenLastTransferredTimestamp(tokenId);	// @audit-issue
```
[46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L46-L46), [50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L50-L50), [119](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L119-L119), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L126-L126), [138](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L138-L138), [142](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L142-L142), [155](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L155-L155), [159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L159-L159), [160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L160-L160), [176](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L176-L176), [187](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L187-L187), [191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L191-L191), 


#### Recommendation

Review your smart contracts to identify and remove `block.number` and `block.timestamp` from the parameters of emitted events. Instead of manually adding these fields, rely on the Ethereum blockchain's inherent inclusion of this information within the block context. Simplify your event definitions to include only the essential data specific to the event's purpose, excluding universally available block metadata.

### Use assembly to check for `address(0)`
In Solidity, it's a common practice to check whether an Ethereum address variable is set to the zero address (`address(0)`) to handle various scenarios, such as token transfers or contract interactions. Typically, this check is performed using a conditional statement like `if (addressVariable == address(0))`.

However, using this approach in high-frequency or gas-sensitive operations can lead to unnecessary gas costs. A more gas-efficient alternative is to use inline assembly to perform the zero address check, which can significantly reduce gas consumption, especially in loops or complex contract logic.

By utilizing inline assembly for this specific check, you can optimize gas usage and make your Solidity code more efficient.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

77:    require(entityForgingAddress_ != address(0), 'Invalid address');	// @audit-issue

85:    require(entropyGeneratorAddress_ != address(0), 'Invalid address');	// @audit-issue

91:    require(airdrop_ != address(0), 'Invalid address');	// @audit-issue

236:      ownerOf(tokenId) != address(0),	// @audit-issue

258:      ownerOf(tokenId) != address(0),	// @audit-issue

268:      ownerOf(tokenId) != address(0),	// @audit-issue
```
[77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L77-L77), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L85-L85), [91](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L91-L91), [236](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L236-L236), [258](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L258-L258), [268](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L268-L268), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

69:    require(listing.seller != address(0), 'NFT is not listed for sale.');	// @audit-issue

113:    require(nukeFundAddress != address(0), 'NukeFund address not set');	// @audit-issue
```
[69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L69-L69), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L113-L113), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

119:    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');	// @audit-issue

138:      nftContract.ownerOf(tokenId) != address(0),	// @audit-issue

187:      nftContract.ownerOf(tokenId) != address(0),	// @audit-issue
```
[119](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L119-L119), [138](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L138-L138), [187](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L187-L187), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `address(0)`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to check for `0`
Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

196:    if (excessPayment > 0) {	// @audit-issue

221:    if (budgetLeft > 0) {	// @audit-issue

277:    return roleIndicator == 0;	// @audit-issue

381:    if (listedId > 0) {	// @audit-issue
```
[196](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L196-L196), [221](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L221-L221), [277](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L277-L277), [381](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L381-L381), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

88:      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,	// @audit-issue

92:    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy	// @audit-issue

137:    require(mergerEntropy % 3 != 0, 'Not merger');	// @audit-issue

141:      mergerForgePotential > 0 &&	// @audit-issue

201:    if (lastForgeResetTimestamp[tokenId] == 0) {	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L88-L88), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L92-L92), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L137-L137), [141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L141-L141), [201](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L201-L201), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

15:    if (totalDevWeight > 0) {	// @audit-issue

19:      if (remaining > 0) {	// @audit-issue

32:    require(weight > 0, 'Invalid weight');	// @audit-issue

33:    require(info.weight == 0, 'Already registered');	// @audit-issue

42:    require(weight > 0, 'Invalid weight');	// @audit-issue

43:    require(info.weight > 0, 'Not dev address');	// @audit-issue

53:    require(info.weight > 0, 'Not dev address');	// @audit-issue

68:    if (pending > 0) {	// @audit-issue
```
[15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L15-L15), [19](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L19-L19), [32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L32-L32), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L33-L33), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L43-L43), [53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L53-L53), [68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L68-L68), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

42:    require(price > 0, 'Price must be greater than zero');	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L42-L42), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

158:    isForger = role == 0;	// @audit-issue

190:    while (number != 0) {	// @audit-issue
```
[158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L158-L158), [190](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L190-L190), 


#### Recommendation

To optimize gas usage in your Solidity code, consider using inline assembly for checking `0`. This approach can significantly reduce gas costs, especially in high-frequency or gas-sensitive operations, leading to more efficient contract execution.

### Use assembly to write `address` storage values
Using assembly `{ sstore(state.slot, addr)}` instead of `state = addr` can save gas.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

69:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

79:    entityForgingContract = IEntityForging(entityForgingAddress_);	// @audit-issue

87:    entropyGenerator = IEntropyGenerator(entropyGeneratorAddress_);	// @audit-issue

93:    airdropContract = IAirdrop(airdrop_);	// @audit-issue

294:    initialOwners[newItemId] = to;	// @audit-issue

329:    initialOwners[newTokenId] = newOwner;	// @audit-issue
```
[69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L69-L69), [79](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L79-L79), [87](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L87-L87), [93](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L93-L93), [294](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L294-L294), [329](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L329-L329), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

26:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

33:    nukeFundAddress = _nukeFundAddress;	// @audit-issue
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L26-L26), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L33-L33), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

23:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

30:    nukeFundAddress = _nukeFundAddress;	// @audit-issue
```
[23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L23-L23), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L30-L30), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

33:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

34:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

35:    devAddress = _devAddress; // Set the developer's address	// @audit-issue

36:    daoAddress = _daoAddress;	// @audit-issue

85:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

90:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

95:    devAddress = account;	// @audit-issue

100:    daoAddress = account;	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L33-L33), [34](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L34-L34), [35](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L35-L35), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L36-L36), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L85-L85), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L90-L90), [95](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L95-L95), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L100-L100), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

31:    allowedCaller = _traitForgetNft;	// @audit-issue

37:    allowedCaller = _allowedCaller;	// @audit-issue
```
[31](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L31-L31), [37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L37-L37), 


#### Recommendation

To reduce gas costs in your Solidity code, consider using assembly with `{ sstore(state.slot, addr) }` for writing `address` storage values instead of `state = addr`. This approach can result in significant gas savings.