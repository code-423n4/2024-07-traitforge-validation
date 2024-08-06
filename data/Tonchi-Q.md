## [Low-1] Locking of `TraitForgeNft::_mintWithBudget` function, breaks the functionality of protocol and noone can mint multiple NFTs.

**Description:** This `TraitForgeNft` contract using `uint256 _tokenIds` variable which informs about how much `_tokenIds` have minted so far. Anytime NFt minted and forged this `_tokenIds` variable updated(\_tokenIds++). In `mintWithBudget` function, if `_tokenIds < maxTokensPerGen` only then Nft is minted. but in entire contract `_tokenIds` variable is not updated anyway except `_tokenIds++`. If `_tokenIds` variable hits `maxTokensPerGen` which is 10000, this mintWithBudget functionality locked.

**Impact:** Noone can mint NFTs with `mintWithBudget` function and can't mint multiple NFTs, breaks functionality of protocol.

**Proof of Concept:**
`_tokenIds` variable is not for maximum token per generation mint counts. The `TraitForgeNft::mintWithBudget` function using \_tokenIds check which should not be there. see in code below:

<details>
<summary>code</summary>

```javascript
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
    uint256 amountMinted = 0;
    uint256 budgetLeft = msg.value;
@>    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
      _mintInternal(msg.sender, mintPrice);
      amountMinted++;
      budgetLeft -= mintPrice;
      mintPrice = calculateMintPrice();
    }
    if (budgetLeft > 0) {
      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
      require(refundSuccess, 'Refund failed.');
    }
  }
```

</details>

**Recommended Mitigation:** Make following changes in code below:

```diff
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
    uint256 amountMinted = 0;
    uint256 budgetLeft = msg.value;
-    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
+    while (budgetLeft >= mintPrice) {
      _mintInternal(msg.sender, mintPrice);
      amountMinted++;
      budgetLeft -= mintPrice;
      mintPrice = calculateMintPrice();
    }
    if (budgetLeft > 0) {
      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
      require(refundSuccess, 'Refund failed.');
    }
  }
```

## [L-2] Wrong path given for imports in openzeppelin contracts

**Description:** In entire scope of this audit, all contracts imported libraries from openzeppelin, in which `ReentrancyGuard.sol` and `Pausable.sol` contracts are inherited and path given is `@openzeppelin/contracts/security/ReentrancyGuard.sol` and `@openzeppelin/contracts/security/Pausable.sol`, but openzeppelin moved these two from security to utils folder.

**Impact:** wrong imports can leads to compilation errors, break in functionality, increased complexity in management.

**Recommended Mitigation:** Use `@openzeppelin/contracts/utils/Pausable.sol` and `@openzeppelin/contracts/utils/ReentrancyGuard.sol` instead.

## [L-3] wrong implementation according to docs in `EntropyGenerator::deriveTokenParameter`

**Description:** In `EntropyGenerator::deriveTokenParameter` function, nukeFactor is calculated as `nukeFactor = entropy / 4000000;` and forgePotential is `forgePotential = getFirstDigit(entropy);`, whereas according to documentation this `nukeFactor` should be `entropy/4` and `forgePotential` should be `5th` digit not 1st.

**Impact:** This `deriveTokenParameter` function returns wrong parameters, misleading information for user.

**Recommended Mitigation:** Correct the calculation for `forgePotential` which should be 5th digit of entropy and `nukeFactor` which should entropy/4.

## [L-4] Missing checks for maximum value, could be manipulate

**Description:** 
- In `DevFund::addDev` function, this `weight` parameter should have Maximum value.
- In `EntityTrading::setTaxCut`, `NukeFund`, `EntityForging`, this `_taxCut` parameter should have maximum value.
  

**Impact:** Because of not maximum value, any value can be set here and it could manipulate funds

**Recommended Mitigation:** apply maximum value checks for these values. 

## [L-5] Weak PRNG, attacker can manipulate randomness and leverage over entropy value positions

**Description:** In `EntropyGenerator::writeEntropyBatch1`, `EntropyGenerator::writeEntropyBatch2`, `EntropyGenerator::writeEntropyBatch3` functions, hash of block.number and i used as a random number but this is not random and attacker can manipulate randomness and leverage over entropy value positions. 

**Impact:** Attacker can take advantage of positions have higher entropy value. Higher entropy value gets higher airdrop and funds while nuking.

**Recommended Mitigation:** Use chainlink VRF as random number generator.
