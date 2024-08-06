# QA Report for **TraitForge**

## Table of Contents

| Issue ID                                                                         | Description                                                      |
| -------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| [QA-01](#qa-01-approved-contracts-cannot-list-or-cancel-nft-sales)               | Approved Contracts Cannot List or Cancel NFT Sales               |
| [QA-02](#qa-02-fix-typos-_multiple-instances_)                                   | Fix Typos _Multiple Instances_                                   |
| [QA-03](#qa-03-remove-stale-information-from-the-whitepaper)                     | Remove Stale Information From The Whitepaper                     |
| [QA-04](<#qa-04-consider-attaching-a-sister-functionality-to-claim/removedev()>) | Consider Attaching A Sister Functionality To `claim/removeDev()` |
| [QA-05](#qa-05-missing-events-for-important-state-changes)                       | Missing Events for Important State Changes                       |
| [QA-06](#qa-06-setter-functions-lack-equality-checks)                            | Setter Functions Lack Equality Checks                            |
| [QA-07](<#qa-07-buynft()-can-be-front-ran>)                                      | buyNFT() Can Be Front-ran                                        |
| [QA-08](#qa-08-lack-of-price-update-functionality-for-listings)                  | Lack of Price Update Functionality for Listings                  |
| [QA-09](#qa-09-inconsistency-of-the-maturity-period)                             | Inconsistency Of The Maturity Period                             |
| [QA-10](#qa-10-insufficient-randomness-in-entropy-generation)                    | Insufficient Randomness in Entropy Generation                    |
| [QA-11](<#qa-11-inefficient-time-conversion-in-nukefund#calculateage()>)         | Inefficient Time Conversion in `NukeFund#calculateAge()`         |
| [QA-12](#qa-12-missing-explicit-maturation-time-range)                           | Missing Explicit Maturation Time Range                           |
| [QA-13](#qa-13-fix-unbounded-tax-cut-setting)                                    | Fix Unbounded Tax Cut Setting                                    |
| [QA-14](#qa-14-lack-of-royalty-mechanism-for-original-creators)                  | Lack of Royalty Mechanism for Original Creators                  |
| [QA-15](<#qa-15-entropy-generation-should-use-blockhash()>)                      | Entropy Generation Should Use `BlockHash()`                      |
| [QA-16](#qa-16-remove-test-codes-prior-to-live-production)                       | Remove Test Codes Prior To Live Production                       |

## QA-01 Approved Contracts Cannot List or Cancel NFT Sales

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L38-L60

```solidity
  function listNFTForSale(
    uint256 tokenId,
    uint256 price
  ) public whenNotPaused nonReentrant {
    require(price > 0, 'Price must be greater than zero');
    require(
      nftContract.ownerOf(tokenId) == msg.sender,
      'Sender must be the NFT owner.'
    );
    require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );

    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract

    ++listingCount;
    listings[listingCount] = Listing(msg.sender, tokenId, price, true);
    listedTokenIds[tokenId] = listingCount;

    emit NFTListed(tokenId, msg.sender, price);
  }
```

And https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L94-L109

```solidity
  function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {
    Listing storage listing = listings[listedTokenIds[tokenId]];

    // check if caller is the seller and listing is acivte
    require(
      listing.seller == msg.sender,
      'Only the seller can canel the listing.'
    );
    require(listing.isActive, 'Listing is not active.');

    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer the nft back to seller

    delete listings[listedTokenIds[tokenId]]; // mark the listing as inactive or delete it

    emit ListingCanceled(tokenId, msg.sender);
  }
```

### Impact

Contracts that are approved to manage NFTs on behalf of their owners cannot list or cancel listings, limiting the flexibility of the system and potentially hindering integration with other smart contracts or platforms.

### Recommended Mitigation Steps

Modify the functions to allow approved contracts to list and cancel:

```solidity
function listNFTForSale(uint256 tokenId, uint256 price) public whenNotPaused nonReentrant {
    require(nftContract.ownerOf(tokenId) == msg.sender || nftContract.getApproved(tokenId) == msg.sender || nftContract.isApprovedForAll(nftContract.ownerOf(tokenId), msg.sender), 'Sender must be the NFT owner or approved.');
    // ... rest of the function ...
}

function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {
    Listing storage listing = listings[listedTokenIds[tokenId]];
    require(listing.seller == msg.sender || nftContract.getApproved(tokenId) == msg.sender || nftContract.isApprovedForAll(listing.seller, msg.sender), 'Only the seller or approved can cancel the listing.');
    // ... rest of the function ...
}
```

This is similar to what has been implemented here https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L153-L157

```solidity
  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {
    require(
      nftContract.isApprovedOrOwner(msg.sender, tokenId),
      'ERC721: caller is not token owner or approved'
    );//@audit
    ..snip
  }
```

## QA-02 Fix Typos _Multiple Instances_

### Proof of Concept

> NB: _Fixes denoted by "\*\* \*\*"_

- https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L97-L101

```diff
-    // check if caller is the seller and listing is acivte
+    // check if caller is the seller and listing is **active**
    require(
      listing.seller == msg.sender,
      'Only the seller can canel the listing.'
    );
```

- https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L151-L155

```diff
-    // example calcualtions using entropyto derive game-related parameters
+    // example calcualtions using **entropy to** derive game-related parameters
    nukeFactor = entropy / 4000000;
    forgePotential = getFirstDigit(entropy);
    performanceFactor = entropy % 10;

```

- https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L197-L203

```diff
-  // utility to get he first digit of a number
+  // utility to get **the** first digit of a number
  function getFirstDigit(uint256 number) private pure returns (uint256) {
    while (number >= 10) {
      number /= 10;
    }
    return number;
  }
```

- https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L135-L136

```diff
-  // function to derive various parameters baed on entrtopy values, demonstrating potential cases
+  // function to derive various parameters baed on **entropy** values, demonstrating potential cases
  function deriveTokenParameters()
```

### Impact

QA

### Recommended Mitigation Steps

```diff
+  Apply the fixes
```

## QA-03 Remove Stale Information From The Whitepaper

### Proof of Concept

This has been stated in the [whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit#heading=h.19r6v8uax1s2):

> Multichain
> The game will be ported to Solana after a launch on ETH L1.

However per the current readMe, the protocol is only to be deployed onn the Base network, which hints one of the docs is stale.

### Impact

QA, confusing integration for all parties.

### Recommended Mitigation Steps

Consider removing stale docs.

## QA-04 Consider Attaching A Sister Functionality To `claim/removeDev()`

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L50-L75

```solidity

  function removeDev(address user) external onlyOwner {
    DevInfo storage info = devInfo[user];
    require(info.weight > 0, 'Not dev address');
    totalDevWeight -= info.weight;
    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;
    info.rewardDebt = totalRewardDebt;
    info.weight = 0;
    emit RemoveDev(user);
  }

  function claim() external whenNotPaused nonReentrant {
    DevInfo storage info = devInfo[msg.sender];

    uint256 pending = info.pendingRewards +
      (totalRewardDebt - info.rewardDebt) *
      info.weight;

    if (pending > 0) {
      uint256 claimedAmount = safeRewardTransfer(msg.sender, pending);
      info.pendingRewards = pending - claimedAmount;
      emit Claim(msg.sender, claimedAmount);
    }

    info.rewardDebt = totalRewardDebt;
  }
```

Both these functions are used to claim and also remove devs by the owner, issue however is that removing a dev does not send them their owned `pendingRewards`.

Note that the removeDev function is a functionality that's expected to be called, this is because a Dev might be eligible for previous rewards based on previous works they've done, but are no longer eligible for future rewards and as such should have the owner remove them so as not get rewards from then.

Issue however is that if/when the dev calls removeDev and the dev hasn't claimed their rewards are always kept for them.

Now whereas the above could be an intended functionality, there need to exist one whereby if this function is called the admin can force the devs to forfeit their rewards to other devs.

### Impact

QA design fix.

### Recommended Mitigation Steps

Consider creating a function that can force the devs to forfeit their rewards to other devs.

## QA-05 Missing Events for Important State Changes

### Proof of Concept

Take a look at the `setNukeFundAddress` and `setTaxCut` functions from the `EntropyTrading.sol`.

```solidity
function setNukeFundAddress(address payable _nukeFundAddress) external onlyOwner {
    nukeFundAddress = _nukeFundAddress;
}

function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
}
```

Also https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L67-L82

```solidity
  function setMinimumDaysHeld(uint256 value) external onlyOwner {
    minimumDaysHeld = value;
  }

  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {
    defaultNukeFactorIncrease = value;
  }

  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {
    maxAllowedClaimDivisor = value;
  }

  function setNukeFactorMaxParam(uint256 value) external onlyOwner {
    nukeFactorMaxParam = value;
  }

```

### Impact

Important state changes are not logged, making it difficult for off-chain services to track and react to these changes. This can lead to inconsistencies between on-chain and off-chain states.

### Recommended Mitigation Steps

Add events for these state changes:

```solidity
event NukeFundAddressChanged(address indexed oldAddress, address indexed newAddress);
event TaxCutChanged(uint256 oldTaxCut, uint256 newTaxCut);

function setNukeFundAddress(address payable _nukeFundAddress) external onlyOwner {
    address oldAddress = nukeFundAddress;
    nukeFundAddress = _nukeFundAddress;
    emit NukeFundAddressChanged(oldAddress, _nukeFundAddress);
}

function setTaxCut(uint256 _taxCut) external onlyOwner {
    uint256 oldTaxCut = taxCut;
    taxCut = _taxCut;
    emit TaxCutChanged(oldTaxCut, _taxCut);
}
```

## QA-06 Setter Functions Lack Equality Checks

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L27-L35

```solidity
  function setNukeFundAddress(
    address payable _nukeFundAddress
  ) external onlyOwner {
    nukeFundAddress = _nukeFundAddress;
  }

  function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
  }
```

Also https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L67-L82

```solidity
  function setMinimumDaysHeld(uint256 value) external onlyOwner {
    minimumDaysHeld = value;
  }

  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {
    defaultNukeFactorIncrease = value;
  }

  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {
    maxAllowedClaimDivisor = value;
  }

  function setNukeFactorMaxParam(uint256 value) external onlyOwner {
    nukeFactorMaxParam = value;
  }

```

### Impact

These functions allow setting values even if they're the same as the current values, potentially wasting gas on unnecessary state changes.

### Recommended Mitigation Steps

Consider adding equality checks to prevent unnecessary state changes:

```solidity
function setNukeFundAddress(address payable _nukeFundAddress) external onlyOwner {
    require(_nukeFundAddress != nukeFundAddress, "New address is the same as current address");
    nukeFundAddress = _nukeFundAddress;
}

function setTaxCut(uint256 _taxCut) external onlyOwner {
    require(_taxCut != taxCut, "New tax cut is the same as current tax cut");
    taxCut = _taxCut;
}
```

## QA-07 buyNFT() Can Be Front-ran

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L63-L92

```solidity
  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
    Listing memory listing = listings[listedTokenIds[tokenId]];
    require(
      msg.value == listing.price,
      'ETH sent does not match the listing price'
    );
    require(listing.seller != address(0), 'NFT is not listed for sale.');

    //transfer eth to seller (distribute to nukefund)
    uint256 nukeFundContribution = msg.value / taxCut;
    uint256 sellerProceeds = msg.value - nukeFundContribution;
    transferToNukeFund(nukeFundContribution); // transfer contribution to nukeFund

    // transfer NFT from contract to buyer
    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(
      ''
    );
    require(success, 'Failed to send to seller');
    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer NFT to the buyer

    delete listings[listedTokenIds[tokenId]]; // remove listing

    emit NFTSold(
      tokenId,
      listing.seller,
      msg.sender,
      msg.value,
      nukeFundContribution
    ); // emit an event for the sale
  }
```

This function is used to buy an NFT listed for sale, however the current implementation allows buyers to front-run each other, potentially leading to unfair purchases where faster or more sophisticated users can snipe desirable NFTs.

### Impact

QA, since this can be argued to be intended functionality.

### Recommended Mitigation Steps

Implement a commit-reveal scheme for purchases:

1. Add a commit phase where buyers submit a hash of their intended purchase.
2. Add a reveal phase where buyers reveal their commitment and complete the purchase.
3. Implement a time delay between phases to prevent immediate reveals.

This approach would make it much harder for front-runners to unfairly snipe NFTs.

## QA-08 Lack of Price Update Functionality for Listings

### Proof of Concept

The contract does not provide a function to update the price of an existing listing. Sellers must cancel and relist to change the price.

### Impact

Sellers are forced to cancel and relist their NFTs to change the price, which is inefficient and could lead to missed sale opportunities during the relisting process. This also increases gas costs for sellers who need to frequently adjust their prices based on market conditions.

### Recommended Mitigation Steps

Implement an `updateListingPrice` function that allows sellers to modify the price of their existing listings:

- https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/IEntityTrading.sol

```diff
+ event ListingPriceUpdated(uint256 indexed tokenId, uint256 newPrice);
```

- https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol

```diff
+ function updateListingPrice(uint256 tokenId, uint256 newPrice) public whenNotPaused nonReentrant {
+     require(newPrice > 0, "Price must be greater than zero");
+     uint256 listingId = listedTokenIds[tokenId];
+     Listing storage listing = listings[listingId];
+     require(listing.seller == msg.sender, "Only the seller can update the listing price");
+     require(listing.isActive, "Listing is not active");
+
+     listing.price = newPrice;
+
+     emit ListingPriceUpdated(tokenId, newPrice);
+ }
+
```

This function allows sellers to update their listing prices without the need to cancel and relist, improving efficiency and user experience.

## QA-09 Inconsistency Of The Maturity Period

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L30

```solidity
uint256 public minimumDaysHeld = 3 days;
```

This is the `minimumDaysHeld` var, now per the [whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit#heading=h.19r6v8uax1s2), this should always be `3` days, however it can be changed by the owner. This would contradict the official documentation stating a fixed 3-day maturity period.

### Impact

QA, deviation from whitepaper implementation.

### Recommended Mitigation Steps

Remove the `setMinimumDaysHeld` function and make `minimumDaysHeld` a constant:

```solidity
uint256 public constant MINIMUM_DAYS_HELD = 3 days;
```

## QA-10 Insufficient Randomness in Entropy Generation

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L56-L60

```solidity
uint256 pseudoRandomValue = uint256(
  keccak256(abi.encodePacked(block.number, i))
) % uint256(10) ** 78; // generate a  pseudo-random value using block number and index
```

This method of generating pseudo-random values is used in all three writeEntropyBatch functions, however the randomness value is gotten in a not so random way

### Impact

Miners can manipulate block.number, making the generated values predictable. Malicious actors could potentially game the system by predicting or influencing the generated entropy.

> Borderline low/med since this could _somewhat_ affect the rarity of an NFT.

### Recommended Mitigation Steps

Consider implementing Chainlink VRF (Verifiable Random Function) for truly random and verifiable off-chain randomness.

## QA-11 Inefficient Time Conversion in `NukeFund#calculateAge()`

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L118-L134

```solidity
  function calculateAge(uint256 tokenId) public view returns (uint256) {
    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');

    uint256 daysOld = (block.timestamp -
      nftContract.getTokenCreationTimestamp(tokenId)) /
      60 /
      60 /
      24;
    uint256 perfomanceFactor = nftContract.getTokenEntropy(tokenId) % 10;

    uint256 age = (daysOld *
      perfomanceFactor *
      MAX_DENOMINATOR *
      ageMultiplier) / 365; // add 5 digits for decimals
    return age;
  }

```

This function is used to calculate the age of a token based on its creation timestamp and current time, whereas it works correctly, its not written in the most efficient manner.

This is because the current implementation uses three separate division operations to convert seconds to days. This approach, while readable, is less gas efficient than performing a single division operation. In a contract where this function might be called frequently, the additional gas cost could accumulate, leading to higher overall transaction costs for users.

### Impact

QA

### Recommended Mitigation Steps

Replace the multiple division operations with a single division by 86400 (the number of seconds in a day):

```diff
  function calculateAge(uint256 tokenId) public view returns (uint256) {
    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');

    uint256 daysOld = (block.timestamp -
-      nftContract.getTokenCreationTimestamp(tokenId)) /
+      nftContract.getTokenCreationTimestamp(tokenId)) / 86400;
-      60 /
-      60 /
-      24;
    uint256 perfomanceFactor = nftContract.getTokenEntropy(tokenId) % 10;

    uint256 age = (daysOld *
      perfomanceFactor *
      MAX_DENOMINATOR *
      ageMultiplier) / 365; // add 5 digits for decimals
    return age;
  }

```

## QA-12 Missing Explicit Maturation Time Range

### Proof of Concept

Take a look at [NukeFund.sol](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol).There's no explicit implementation of a 30-600 day maturation range, however per the [whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit#heading=h.19r6v8uax1s2) this should be the only valid range.

### Impact

QA, deviation from whitepaper implementation.

### Recommended Mitigation Steps

Do not deviate from the whitepaper or fix whitepaper if stale.

## QA-13 Fix Unbounded Tax Cut Setting

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L33-L35

```solidity
  function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
  }
```

### Impact

The tax cut can be set to any value, including 0 or extremely high values. This could lead to unexpected behavior, such as division by zero errors or the entire sale price being taken as tax.

### Recommended Mitigation Steps

Add bounds checking to the `setTaxCut` function:

```solidity
function setTaxCut(uint256 _taxCut) external onlyOwner {
    require(_taxCut > 0 && _taxCut <= 100, "Tax cut must be between 1 and 100");
    taxCut = _taxCut;
}
```

## QA-14 Lack of Royalty Mechanism for Original Creators

### Proof of Concept

Take a look at [EntityTrading.sol](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol), it's evident that there's no implementation of a royalty mechanism.

### Impact

Original creators of NFTs are not compensated when their creations are resold on this marketplace. This could discourage artists and creators from using the platform, potentially limiting the variety and quality of NFTs available.

### Recommended Mitigation Steps

Implement EIP-2981 (NFT Royalty Standard) in the contract. Add a royalty calculation in the `buyNFT` function and distribute the royalties to the original creator along with the seller's proceeds.

```solidity
function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
    // ... existing code ...

    (address royaltyReceiver, uint256 royaltyAmount) = nftContract.royaltyInfo(tokenId, msg.value);
    uint256 sellerProceeds = msg.value - nukeFundContribution - royaltyAmount;

    // Transfer royalties
    if (royaltyAmount > 0) {
        (bool royaltySuccess, ) = payable(royaltyReceiver).call{value: royaltyAmount}("");
        require(royaltySuccess, "Failed to send royalties");
    }

    // ... rest of the function ...
}
```

## QA-15 Entropy Generation Should Use `BlockHas()`

### Proof of Concept

Take a look at https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L56-L60

```solidity
uint256 pseudoRandomValue = uint256(
  keccak256(abi.encodePacked(block.number, i))
) % uint256(10) ** 78;
```

The contract uses `block.number` and an index for entropy generation, while the [whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit#heading=h.19r6v8uax1s2) states that **"Genesis entropy is seeded using blockhash"**. This inconsistency could lead to less secure/random entropy generation and also shows how not the intended functionality is being followed.

### Impact

QA, albeit borderline low/med since intended functionality is not being followed.

### Recommended Mitigation Steps

Consider using a logic like the below:

```solidity
uint256 pseudoRandomValue = uint256(
  keccak256(abi.encodePacked(blockhash(block.number - 1), i))
) % uint256(10) ** 78;
```

## QA-16 Remove Test Codes Prior To Live Production

### Proof of Concept

Multiple instances in scope we see helper vars/functionalities and what not that are meant to be for testing purposes still existing in code, for e.g see https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L24-L25

```solidity
  uint256 public ageMultiplier;

```

Also https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L109-L115

```solidity
  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {
    ageMultiplier = _ageMultiplier;
  }

  function getAgeMultiplier() public view returns (uint256) {
    return ageMultiplier;
  }
```

The `ageMultiplier` is completely of no use to the protocol in live production and as such should be removed.

### Impact

QA

### Recommended Mitigation Steps

Apply these changes:

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L11-L196

```diff
contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {
  uint256 public constant MAX_DENOMINATOR = 100000;
..snip
-  uint256 public ageMultiplier;
..snip


-  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {
-    ageMultiplier = _ageMultiplier;
-  }
-
-  function getAgeMultiplier() public view returns (uint256) {
-    return ageMultiplier;
-  }
..snip
```
