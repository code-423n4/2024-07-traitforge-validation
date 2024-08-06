# [QA-O1] Typos and Misleading Error Messages
## First instance:
## Description 
The function `setAgeMultplier` contains a typo in its name. The correct term should be `Multiplier` instead of `Multplier`. 
## Impact
This inconsistency can cause confusion and make the contract harder to maintain.
## Recommendation
Rename the Function: Update the function name from `setAgeMultplier` to `setAgeMultiplier` to correct the typo and ensure clarity.

## Second instance:
## Description 
The `setMaxGeneration` function within the `TraitForgeNft` contract contains a require statement designed to enforce that the new maximum generation value (`maxGeneration_`) is not less than the current generation (`currentGeneration`). However, the error message associated with this require statement is grammatically incorrect and misleading.
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L117-L118
## Impact
The error message `"can't below than current generation"` does not accurately describe the condition being checked, leading to potential confusion among developers or users attempting to debug or understand the contract's behavior
## Recommendation
`"can't be less than current generation"` or `"can't be below current generation"`

## Third instance:
## Description
The word "cancel" is misspelled as "canel". 
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L100-L101
## Recommendation 
'Only the seller can cancel the listing'

# [QA-O2] Potential for Griefing Attacks in DevFund's `receive()` Function
## Description 
The `receive()` function in the DevFund contract is vulnerable to potential griefing attacks. An attacker can exploit this function by sending multiple transactions with minimal ETH amounts (even as low as 1 wei)
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L14-L28
## Impact
- Precision loss in reward calculations due to frequent small-value divisions.
- Potential Denial of Service (DoS) on the owner's ability to receive funds if the owner is a contract with complex logic.
- Event log spam, making it difficult to track legitimate transactions.
- Frequent state changes, increasing gas costs for legitimate users.
## Recommendation
- Introduce a minimum contribution amount
- Implement a cooldown period between contributions from the same address

# [QA-O3] Incomplete Listing Removal 
## First instance:
## Description 
The `_cancelListingForForging` function in the `EntityForging` contract does not fully remove all references to a cancelled listing. While it deletes the listing from the `listings` mapping, it fails to remove the corresponding entry from the `listedTokenIds` mapping.
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L193-L197
## Impact
- Data inconsistency: The `listedTokenIds` mapping retains a reference to a non-existent listing.
- Potential for errors: Future operations involving the same token ID may reference an incorrect or non-existent listing.
## Recommendation
Modify the `_cancelListingForForging` function to remove entries from both mappings
## Second instance:
## Description
The `cancelListing` function in the `EntityTrading` contract does not fully remove all references to a cancelled listing. While it deletes the listing from the `listings` mapping, it fails to remove the corresponding entry from the `listedTokenIds` mapping.
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L94-L109
## Impact
- Data Inconsistency: The `listedTokenIds` mapping retains a reference to a non-existent listing, leading to inconsistent contract state.
- Potential for Errors: Future operations involving the same token ID may reference an incorrect or non-existent listing, potentially causing unexpected behavior or errors
## Recommendation
Modify the `cancelListing` function to remove entries from both mappings

# [QA-O4] Potential DoS with Block Gas Limit in `fetchListings` Function
## Description 
The `fetchListings` function in the EntityForging contract attempts to return all active listings in a single call. As the number of listings grows, the gas required to execute this function increases. If the number of listings becomes too large, the gas required to execute this function could exceed the block gas limit, making it impossible to call this function. This can lead to a Denial of Service (DoS) situation where users or other contracts cannot retrieve listing data.
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L48-L54
## Impact
- Unusable Function: If the number of listings grows too large, this function will become unusable, preventing users or other contracts from retrieving listing data.
- Potential Contract Lockup: If other critical functions rely on `fetchListings`, they could also become unusable, potentially locking up parts of the contract's functionality.
## Recommendation
Implement pagination in the `fetchListings` function to allow callers to request a specific range of listings, preventing the function from attempting to return too much data at once.