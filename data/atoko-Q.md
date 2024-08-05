
## Low Risk Findings

| Number | issue                                                                                                   |
| ------ | ------------------------------------------------------------------------------------------------------- |
| [L-01] | Pause/Unpause functionalities are not implemented in many pausable contracts                                           |
| [L-02] | Inadequate Handling of Mappings                                                  |
| [L-03] | Inconsistent Listing Count Management                                                           |
| [L-04] | contract approvals not revoked                                            |
| [L-05] | Inability to Set Approval for Address                                                                 |
| [L-06] | Missing Active Listing Check in buyNFT Function                                                                     |
| [L-07] | Consider Using Chainlink for Randomness |
| [L-08] | Division Before Multiplication                                                              |
| [L-09] | Sellers not prevented from buying their own NFTs                                                                        |
| [L-10] | Listings cannot be updated                                                                                  |
| [L-11] | Activate the Optimizer             |
| [L-12] | Off by one error                                                 |
| [L-13] | Low level calls to account with no code will not fail                                                |

## [L-01] Pause/Unpause functionalities are not implemented in many pausable contracts

In the current implementation of the protocol's contracts, which are intended to be pausable by design (as they inherit from the Pausable contract provided by OpenZeppelin), there is a critical oversight. While these contracts do include internal functions _pause and _unpause since they inherit from pausable, they lack the necessary public or external functions that allow a contract manager or administrator to actually trigger these pause and unpause operations.
```javascript
contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {}
contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable{}
contract EntityTrading is IEntityTrading, ReentrancyGuard, Ownable, Pausable{}
contract TraitForgeNft is ITraitForgeNft, ERC721Enumerable, ReentrancyGuard, Ownable, Pausable{}

```
As it is shown above all the contracts are meant to be pausable but because those functions in the pausable are internal, the contract must implement two other public/external pause and unpause functions to allow the manager to pause and unpause the contracts when necessary. None of the aforementioned contracts implement those functions, which means even if those contracts are supposed to be pausable (and have the pause/unpause functionalities), none of them can be paused.
without the corresponding public or external functions to expose these internal mechanisms, the contracts remain permanently in their operational state and cannot be paused when necessary.

## Recommended Mitigation Steps
Add public/external pause and unpause functions in the aforementioned contracts to allow them to be pausable, this can be done as in the UserWithdrawalManager contract. For example:

```javascript
/**
 * @dev Triggers stopped state.
 * Contract must not be paused
 */
function pause() external {
    UtilLib.onlyManagerRole(msg.sender, staderConfig);
    _pause();
}

/**
 * @dev Returns to normal state.
 * Contract must be paused
 */
function unpause() external onlyRole(DEFAULT_ADMIN_ROLE) {
    _unpause();
}
```

///////////////

## [L-02] Inadequate Handling of Mappings

Summary
The `cancelListingForForging` function in the current contract implementation is responsible for canceling the listing of a token, but it does not adequately update or clear the mappings related to token listings. Specifically, the function interacts with the listedTokenIds and listings mappings, but fails to manage these mappings correctly when a listing is canceled.

The cancelListingForForging function performs the following steps:

* The function verifies that the caller is either the owner of the token or the contract itself, ensuring that only authorized parties can cancel a listing.

* The function checks whether the token is currently listed by examining the isListed flag in the listings mapping.

* The function then invokes _cancelListingForForging(tokenId), which is intended to handle the actual cancellation of the listing.

While these steps are correctly implemented, there is a critical oversight in managing the mappings used to track token listings.

```javascript
function cancelListingForForging(
    uint256 tokenId
  ) external whenNotPaused nonReentrant {
    require(
      nftContract.ownerOf(tokenId) == msg.sender ||
        msg.sender == address(nftContract),
      'Caller must own the token'
    );
    require(
      listings[listedTokenIds[tokenId]].isListed,
      'Token not listed for forging'
    );

    _cancelListingForForging(tokenId);
  }
```

```javascript
mapping(uint256 => uint256) public listedTokenIds;

```
This mapping associates a tokenId with an index used to retrieve the corresponding listing information.

The current implementation does not update or delete the listedTokenIds mapping when a listing is canceled. As a result, the tokenId to index association remains intact, even though the token is no longer listed. This could lead to stale or incorrect listing data being retained.

## This is also a problem when `forgeWithListed` is called as it does not delete the mapping correctly

* Instances

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L194
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L106
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L83

## recommendation

When a listing is canceled, the mapping should be cleared or updated to reflect that the token is no longer listed. This ensures that the tokenId to index association does not persist unnecessarily.

//////////////////

## [L-03] Inconsistent Listing Count Management

The listingCount is incremented whenever a new listing is created through the listForForging function but is not decremented when a listing is canceled or utilized. This discrepancy can lead to significant issues in listing management, data integrity, and overall contract functionality.

```javascript
function listForForging(
    uint256 tokenId,
    uint256 fee
  ) public whenNotPaused nonReentrant {
    Listing memory _listingInfo = listings[listedTokenIds[tokenId]];

    require(!_listingInfo.isListed, 'Token is already listed for forging');
    require(
      nftContract.ownerOf(tokenId) == msg.sender,
      'Caller must own the token'
    );
    require(
      fee >= minimumListFee,// @audit what if this fee is updated
      'Fee should be higher than minimum listing fee'
    );

    _resetForgingCountIfNeeded(tokenId);

    uint256 entropy = nftContract.getTokenEntropy(tokenId); // Retrieve entropy for tokenId
    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy
    require(
      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
      'Entity has reached its forging limit'
    );

    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy
    require(isForger, 'Only forgers can list for forging');

@>>>    ++listingCount;
    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);
    listedTokenIds[tokenId] = listingCount;

    emit ListedForForging(tokenId, fee);
  }
```

## Description

The `listedTokenIds` mapping associates a tokenId with an index, which is essentially the listingCount. Without proper decrementing, this index mapping may contain invalid or outdated entries, causing difficulties in accessing and managing listings.

```javascript
mapping(uint256 => uint256) public listedTokenIds;
```

The `listingCount` variable is intended to track the number of active listings within the contract. it is incremented each time a new listing is created. However, there is no mechanism to decrement this count when a listing is canceled or used. If the listingCount is not decremented correctly, it will not accurately reflect the number of active listings. This can result in stale or incorrect data, leading to mismanagement of listings and potential errors.

`Other Instances`

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L94
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L63

## Recommendation
Ensure Consistent Listing Count Management in All Relevant Functions:

```javascript
function _cancelListingForForging(uint256 tokenId) internal {
    uint256 index = listedTokenIds[tokenId];
    delete listings[index];
    delete listedTokenIds[tokenId];

    // Decrement listingCount
 @>>   listingCount--;

    emit CancelledListingForForging(tokenId);
}

```
This should also be done in the `forgeWithListed` function

/////////////////

## [L-04] contract approvals not revoked

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L48

The `EntityTrading` fails to address the revocation of the contract’s approval to transfer NFTs after they have been listed for sale or after a sale is completed. This oversight could lead to significant security risks and operational inefficiencies.

## Detailed Description

When an NFT is listed for sale using the `listNFTForSale` function, the contract requires the owner to approve it for transfer. This approval allows the contract to move the NFT from the owner's account to itself.

```javascript
 require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );
```
Similarly, when an NFT is bought using the buyNFT function or the listing is canceled using the cancelListing function, the NFT is transferred, but there is no mechanism to revoke the approval granted to the contract.

If the contract retains approval to transfer the NFT after a sale or cancellation, this could pose security risks if a malicious user takes control of the contract. The contract could be exploited to transfer NFTs without proper authorization, especially if the contract or its state is compromised.

## Recommendation

Consider revoking approval of contract after sale or when calling `cancelListing`

///////////////////

## [L-05] Inability to Set Approval for Address

In the codebase,  there dont seem to be a way that users are able to set approval for the contract address or addresses to manage their NFTs. This issue stems from the absence of functions in the nftContract that facilitate approval, specifically the setApproval functions. As a result, the `listNFTForSale` function will fail, preventing users from listing their NFTs for sale. The `listNFTForSale` function requires that the contract address be approved to transfer the NFT on behalf of the owner. 
Without these functions to set approval, users cannot grant the contract address the necessary approval to transfer their NFTs,

Since the listNFTForSale function checks if the contract address has approval to transfer the NFT (nftContract.getApproved(tokenId) == address(this) or nftContract.isApprovedForAll(msg.sender, address(this))), the absence of these functions means the check will always fail. Consequently, users will be unable to list their NFTs for sale through this function.

```javascript
require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );
```
## Recommendations
Add functions that will allow users to setApprovals to addresses they like

////////////////

## [L-06] Missing Active Listing Check in buyNFT Function

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L63

The buyNFT function does not include a verification step to ensure that the NFT listing is active before proceeding with the purchase. This oversight could result in various issues, including allowing the purchase of NFTs that are no longer available for sale.

```javascript
function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
  Listing memory listing = listings[listedTokenIds[tokenId]];
  require(
    msg.value == listing.price,
    'ETH sent does not match the listing price'
  );
  require(listing.seller != address(0), 'NFT is not listed for sale.');

  // Transfer eth to seller (distribute to nukefund)
  uint256 nukeFundContribution = msg.value / taxCut;
  uint256 sellerProceeds = msg.value - nukeFundContribution;
  transferToNukeFund(nukeFundContribution); // Transfer contribution to nukeFund

  // Transfer NFT from contract to buyer
  (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(
    ''
  );
  require(success, 'Failed to send to seller');
  nftContract.transferFrom(address(this), msg.sender, tokenId); // Transfer NFT to the buyer

  delete listings[listedTokenIds[tokenId]]; // Remove listing

  emit NFTSold(
    tokenId,
    listing.seller,
    msg.sender,
    msg.value,
    nukeFundContribution
  ); // Emit an event for the sale
}

```
## Issue Description:
The buyNFT function checks whether the NFT is listed for sale by verifying if the seller address is not zero. However, it does not validate whether the listing is active `(listing.isActive)`. This means that a user could potentially purchase an NFT that was previously listed but has since been canceled or marked inactive.

Users may end up buying NFTs that should no longer be available for sale.

## Recommendation

Update the buyNFT function to include a verification step to ensure that the listing is active before proceeding with the purchase.

```javascript
require(listing.isActive, 'Listing is not active.'); 
```

/////////////

## [L-07] Consider Using Chainlink for Randomness

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L9

The current implementation does not utilize Chainlink VRF (Verifiable Random Function) for generating randomness. Incorporating Chainlink VRF can significantly enhance the security and fairness of randomness used in the contract, particularly for calculating entropy.

The contract currently relies on internal or less secure methods for generating randomness, which could be susceptible to manipulation or predictability. This poses risks to the fairness and integrity of operations dependent on random values, such as entropy calculations.

Chainlink VRF provides cryptographic proofs that ensure the randomness is generated in a secure and tamper-proof manner. By integrating Chainlink VRF, the randomness used for entropy calculations will be protected from manipulation, guaranteeing that the outcomes are truly random and fair.

given that Entropy is a measure of randomness that plays a crucial role in various calculations and processes within the contract. By using Chainlink VRF to generate the randomness for entropy calculations, the contract will benefit from higher quality randomness, reducing the risk of biased or predictable outcomes. This leads to more reliable and fair results in features such as token minting, forging, or other operations dependent on random values.

## Recommendation
Update the contract to incorporate Chainlink VRF for all randomness requirements. This involves requesting randomness from Chainlink VRF and incorporating the returned values into entropy calculations and other relevant operations.

/////////////////

## [L-08] Division Before Multiplication

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L121

In the calculateAge function, there is a precision issue caused by performing division before multiplication. This can result in inaccuracies in the final age calculation for NFTs. This is because multiplication is done on a result that was achived after some division

Detailed Explanation
The function calculates the age of an NFT based on the number of days since its creation and a performance factor derived from its entropy. The calculation is performed using the following formula:

```javascript
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

```javascript
uint256 age = (daysOld * perfomanceFactor * MAX_DENOMINATOR * ageMultiplier) / 365;
```
as we can see the daysOld contains some divisions then they use daysOld in some multiplications

The number of days since the NFT was created is calculated by dividing the time difference by the number of seconds in a day.
The days are multiplied by perfomanceFactor, MAX_DENOMINATOR, and ageMultiplier, and then divided by 365 to adjust for a yearly average.

The problem with this approach is that the division is performed after the multiplication. Specifically, the result of the multiplication is divided by 365. If the intermediate result of the multiplication is large, division before multiplication can lead to truncation or rounding errors. This is because division can result in a loss of precision when applied to large numbers, which impacts the accuracy of the final result.

## Recommendation
Use math libraries to prevent any precision issues

///////////////////

## [L-09] Sellers not prevented from buying their own NFTs

The current implementation of the listNFTForSale and buyNFT functions allows sellers to potentially purchase their own listed NFTs. This creates a loophole that could be exploited by malicious users to gain undue benefits, such as airdrops or other incentives tied to NFT ownership.

## Details

```javascript
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

  nftContract.transferFrom(msg.sender, address(this), tokenId); // transfer NFT to contract

  ++listingCount;
  listings[listingCount] = Listing(msg.sender, tokenId, price, true);
  listedTokenIds[tokenId] = listingCount;

  emit NFTListed(tokenId, msg.sender, price);
}

```

buy

```javascript
function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
  Listing memory listing = listings[listedTokenIds[tokenId]];
  require(
    msg.value == listing.price,
    'ETH sent does not match the listing price'
  );
  require(listing.seller != address(0), 'NFT is not listed for sale.');

  // transfer eth to seller (distribute to nukefund)
  uint256 nukeFundContribution = msg.value / taxCut;
  uint256 sellerProceeds = msg.value - nukeFundContribution;
  transferToNukeFund(nukeFundContribution); // transfer contribution to nukeFund

  // transfer NFT from contract to buyer
  (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }( //@audit seller can receive and burn
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

In the current implementation, there is no mechanism to prevent a seller from also acting as the buyer of their listed NFT.
a seller might exploit this loophole by buying their own NFT to receive these benefits multiple times or circumvent restrictions. this can also lead to artificial inflation

## Recommendation
 implement a check that prevents sellers from being able to purchase their own listed NFTs. 

 ```javascript
 require(listing.seller != msg.sender, 'Seller cannot buy their own NFT');

 ```

 ///////////////////

 ## [L-10] Listings cannot be updated

 The current implementation provides a mechanism for listing NFTs for sale and canceling these listings, but it lacks a functionality for updating existing listings. Specifically, once an NFT is listed for sale, users can only cancel the listing, but cannot modify the price or other relevant details without first canceling and re-listing the NFT.

 ```javascript
 // Function to list NFT for sale
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

    nftContract.transferFrom(msg.sender, address(this), tokenId); // Transfer NFT to contract

    ++listingCount;
    listings[listingCount] = Listing(msg.sender, tokenId, price, true);
    listedTokenIds[tokenId] = listingCount;

    emit NFTListed(tokenId, msg.sender, price);
}

 ```

 The current system allows users to list an NFT for sale and cancel the listing if needed. However, it lacks a function to update the listing details, such as changing the price, without first canceling the existing listing and creating a new one.

  Users who wish to adjust the sale price of their listed NFT are required to cancel the existing listing and then relist the NFT with the new price. This can be inefficient and may involve additional gas costs for multiple transactions.

  ## Recommendation
  Introduce a function that allows users to update the price or other details of an existing listing. This function should ensure that only the owner of the listing can make updates and should verify that the listing exists and is active.

  ///////////////

  ## [L-11] Activate the Optimizer

Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

module.exports = {
solidity: {
version: "0.8.24",
settings: {
optimizer: {
enabled: true,
runs: 1000,
},
},
},
};
Please visit this site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past.

A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

//////////////////

## [L-12] Off by one error

Using the <= operator for time comparisons against block.timestamp can introduce off-by-one errors due to the nature of how block.timestamp is updated only once per block. This can lead to unexpected behavior if the condition is met at the exact second when block.timestamp changes. This issue is especially critical in scenarios where time-sensitive operations are performed, potentially causing operations to revert unexpectedly or execute when they shouldn't.

Proof of Concept (PoC)
Consider the following code snippet:

```javascript
  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
    if (block.timestamp <= whitelistEndTime) {
      require(
        MerkleProof.verify(proof, rootHash, leaf),
        'Not whitelisted user'
      );
    }
    _;
```

In this code, the condition checks if the current block.timestamp is less than or equal to `whitelistEndTime`. If the condition is true, it returns 0

In Solidity, using >= or <= to compare against block.timestamp (alias now) may introduce off-by-one errors due to the fact that block.timestamp is only updated once per block and its value remains constant throughout the block's execution. If an operation happens at the exact second when block.timestamp changes, it could result in unexpected behavior. To avoid this, it's safer to use strict inequality operators (> or <). For instance, if a condition should only be met after a certain time, use block.timestamp > time rather than block.timestamp >= time. This way, potential off-by-one errors due to the exact timing of block mining are mitigated, leading to safer, more predictable contract behavior.

**_Instances_**

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L52

similar issue was reported in the euler contest



Recommendation
To avoid potential off-by-one errors, use strict inequality operators (> or <) instead of non-strict ones (>= or <=). This ensures that the condition only triggers after the specified time has completely passed.

```javascript
  if (block.timestamp < whitelistEndTime) {
      require(
        MerkleProof.verify(proof, rootHash, leaf),
        'Not whitelisted user'
      );
```

////////////////////

## [L-13] : Low level calls to account with no code will not fail 

Low level calls to account with no code will not fail

Low level calls `(i.e. address.call(...))` to account with no code will silently succeed without reverting or throwing any error. Quoting the reference for the CALL opcode in evm.codes:

Creates a new sub context and execute the code of the given account, then resumes the current one. Note that an account with no code will return success as true.

***Instances***
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L361
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L222
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L222
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L51
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L47




