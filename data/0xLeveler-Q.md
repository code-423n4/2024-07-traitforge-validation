# Quality Assurance Report for TraitForgeNFT

The `TraitForge protocol` is a competitive NFT ecosystem designed for the creating and managing evolution of unique NFTs(entities) across multiple generations. At it's core, it features the TraitForgeNft contract, which allows users to mint, burn, and forge NFTs with dynamic pricing and generation-based attributes. The protocol integrates various external contracts for entropy generation, entity forging, and airdrop management, ensuring a secure and fair distribution of assets. It employs security mechanisms, including whitelisting with Merkle proofs, reentrancy guards, and pausability, to protect user interactions. Additionally, the protocol supports a sophisticated fund distribution system that channels proceeds to a designated nuke fund, promoting sustainable development and rewarding contributors. Overall, TraitForge offers a robust and scalable framework for creating and evolving NFTs in a decentralized manner.

The `strategies` for the game fall into the following diagrammed categories:

![TraitForge Strategies](https://i.imgur.com/WvtJ83N.png)

The rest of the project was subsequently analyzed for Quality Assurance using the following methodology which is outlined per contract code.

## Methodology

![methodology](https://i.imgur.com/3lepwE9.png)

# DevFund.sol 



## Contract Summary
The DevFund contract is designed to distribute Ether rewards to developers based on assigned weights. It allows the contract owner to add, update, and remove developers, with a input weight that determines their share of the rewards. When Ether is sent to the contract, it calculates the amount per weight unit and updates the total reward debt accordingly, distributing any remaining Ether to the owner. Developers can claim their pending rewards, which are calculated based on the difference between the total reward debt and their individual reward debt, multiplied by their weight. The contract includes security features such as reentrancy protection, pausability, and safe Ether transfers.

![DevFund Contract](https://i.imgur.com/GSEA2ri.png)

![DevFund Function flow](https://i.imgur.com/9ZZXpMk.png)

## Questions and suggested Improvements

### Questions
- is there sufficient input validation for entered values ?
- are the state changes and updates consistent and in line with contract design ?
- is there a rounding error during `totalRewardDebt` calculation ?
- what are the state changes around transfer of value
- can removed Dev still claim rewards ?
- can updating dev break core protocol invariants ?
- can rewards be stolen ?
- can rewards be front-run or bricked ?
- are `pendingRewards` calculated correctly

### Suggested Improvements
- Emitting better detailed events
- seperation of code storage to reduce cluster
- using OpenZeppelin's `safeTransfer` for  token transfers

  

## Low and Non-critical Vulnerabilities

### [L-01] Possible race-condition, if user can see the mempool and tries to withdraw all their rewards before other users

### [NC-01] Consider using openZeppelin `safeTransfer` module in place of low level call method

### [NC-02] Consider seperating contract storage for code readability

### [L-01] Centralization risks may lead to low trust and low investment

### [NC-03] better error handling using custom errors instead of require statements for gas savings too







# traitForgeNFT.sol

## Contract Summary

![TraitforgeNFT function Flow](https://i.imgur.com/O1dH50W.png) 

This contract is an ERC721-compliant NFT contract that supports minting, burning, and forging (breeding) of tokens across multiple generations. It incorporates features such as whitelisting using Merkle proofs, dynamic pricing based on the number of tokens minted, and an incrementing price mechanism for each new generation. The contract interacts with external contracts for entropy generation, entity forging, and airdrops, and includes security measures like reentrancy guards, pausability, and ownership controls. Additionally, it manages token metadata, including creation timestamps, entropy values, and transfer timestamps, while ensuring proper fund distribution to a designated nuke fund address.

![Traitforge Strategies](https://i.imgur.com/bG9GSYU.png)



### Questions 
- is it possible for 10th generation tokens to forge ?
- is strict equality in `mintToken` an attack vector ?
- is there any incongruence in state updates for `entropy` and `tokenIds` ?
- is the `mintPrice` calculated correctly ?
- will NFTs sent to Smart Contracts be lost ? 
- what happens to the entropy of a token that is burnt ?
- can malfunctions in the external calls cause denial of service ?
- should there be any extra checks in before Token transfer ?

### suggested Improvements 

- central state updates in case of 
- use safeTransfer incase caller is a contract address
- variables that can change core protocol vulnerabilities should only be updated when contracts are paused
- centralization risks


## Low and Non-critical Vulnerabilities
 
### [L-01] use Openzeppelin safeTransfer instead of call

### [NC- 01] caching of storage Variables in memory wheneve necessary

### [NC- 01] centralization risk, owner controlled functions could be multisig







# EntityForging.sol

## Contract Summary

This contract is a Solidity-based smart contract facilitates the listing and forging of NFTs, is uses OpenZeppelin library for security and access control. It allows NFT owners to list their tokens for forging, set various parameters like tax cut and minimum listing fee, and manage the forging process, including fee distribution and potential resets. The contract ensures only eligible tokens can be listed or used for forging, tracks forging counts, and integrates with an external NFT contract (ITraitForgeNft) to handle token ownership and entropy-based logic. Key features include non-reentrancy, pausability, and owner-specific functions for configuration.

![Traitforge Strategies](https://i.imgur.com/IXq4mx1.png)

Lack of Fallback Function for ETH Recovery
The contract has payable functions but lacks a fallback function for receiving ETH directly. This could lead to ETH being sent to the contract accidentally and being irrecoverable if not handled properly

Insufficient Input Sanitization
Ensure that all user inputs are properly sanitized and validated to prevent unexpected behavior or exploits. For instance, checks in the constructor and function parameters should ensure that valid addresses and values are provided.


### Questions 
- can a token be listed and traded ?
- can buyers avoid paying forging fees ?
- are mappings correctly updated and cleared ?
- are the fees (e.g., minimumListFee, taxCut) reasonable and correctly implemented?
- is the fee distribution logic (e.g., devFee, forgerShare) accurate and fair?
- Does the listForForging function correctly handle the listing process?
- Are the conditions for listing and forging tokens logical and secure?
- Is the forgeWithListed function correctly handling the forging process and fee distribution?
- Are the forgingCounts and lastForgeResetTimestamp mappings correctly managed and reset?

### suggested Improvements 

- Consider improving Error Messages: Provide more descriptive error messages.
- Optimize Gas Usage: Review and optimize for gas efficiency.

## Low and Non-critical Vulnerabilities

### [NC- 01] caching of storage Variables in memory wheneve necessary

### [NC- 01] functions that consume too much gas could be refactored to favor users



# EntityTrading.sol 
## Contract Summary

This contract is used for trading NFTs, and compliant with the ERC721 standard. It allows users to list NFTs for sale, buy listed NFTs, and cancel listings. The contract includes features such as a tax cut on sales, which is sent to a designated "NukeFund" address, and integrates security measures from OpenZeppelin like ReentrancyGuard, Ownable, and Pausable. It interacts with an external NFT contract (TraitForgeNft) to manage ownership and transfers of NFTs.

![Entity Trading Function Flow](https://i.imgur.com/7EhZhAo.png)



### Questions 
- Are there any reentrancy vulnerabilities?
- Is the contract protected against integer overflow/underflow?
- Are there any unchecked external calls?
- Is the onlyOwner modifier correctly applied to sensitive functions?

- Does the contract correctly handle the listing, buying, and canceling of NFTs?
- Are the events emitted correctly for each action?
- Is the transferToNukeFund function secure and correctly implemented?

- Are the onlyOwner and whenNotPaused modifiers used appropriately?
Is the nukeFundAddress set and used correctly?

- Does the contract comply with the ERC721 standard?
- Are the imported OpenZeppelin contracts up-to-date and correctly used

- What happens if a user tries to list an NFT they don't own?
- What if the nukeFundAddress is not set before a sale?
- How does the contract handle extremely high or low tax rates?

### suggested Improvements 

- Add NatSpec comments for functions and state variables for better documentation.

- Consider making listingCount a private variable if it's not intended to be accessed externally.


- Validate the _traitForgeNft address to ensure it's not a zero address.
setNukeFundAddress Function:


- Add a check to ensure the listing is active before canceling.
Consider adding a time lock to prevent immediate re-listing after cancelation.

## Low and Non-critical Vulnerabilities

### [L -01] Add validation to ensure _nukeFundAddress is not a zero address.

### [L-02] Add a constant keyword to taxCut if it's not meant to be changed after deployment.

# NukeFund.sol


## Contract Summary

This contract manages a fund that can be claimed by burning NFTs, distributing a portion of received ETH to developer and DAO addresses. It interacts with ITraitForgeNft for NFT operations and IAirdrop for airdrop conditions, with key functions including receiving ETH, calculating NFT age and nuke factors, and handling fund claims through the nuke function. Admin functions allow setting various parameters and updating critical addresses, while security is enforced through modifiers like nonReentrant, onlyOwner, and whenNotPaused.

![Entity Trading Function Flow](https://i.imgur.com/c4TJfKC.png)



### Questions 
- Are there any potential reentrancy vulnerabilities despite the use of nonReentrant?

- Are onlyOwner functions properly restricted and secure?
Is ownership management robust?

- Are external calls in receive and nuke functions safe and properly handled?

- Are there any risks of fund mismanagement or incorrect balance updates?

- Are the methods from nftContract (e.g., ownerOf, getTokenCreationTimestamp) secure and correctly implemented?

- Is the whenNotPaused modifier used appropriately?

- Does the receive function handle unexpected ETH transfers securely?


### suggested Improvements 

- Ensure all external calls are safe, even with the nonReentrant modifier.

- Implement a multi-signature wallet for critical functions to enhance security.

- Add checks to prevent overflows/underflows in fund calculations.
Consider using OpenZeppelin's SafeMath library for safer arithmetic operations.

- Add more comprehensive checks to ensure nftContract methods are secure and correctly implemented.

- Ensure the pause functionality is tested thoroughly and can be invoked in emergency situations.

- Add a fallback function to handle unexpected data and ETH transfers securely.

## Low and Non-critical Vulnerabilities

### [L-01] Use safeTransfer instead of call for ETH transfers to prevent reentrancy and ensure gas stipend.

### [NC-01] Implement a multi-signature wallet for critical functions to enhance security.

# EntropyGenerator.sol
This contract generates pseudo-random values for use in other contracts, leveraging block data for randomness. It initializes entropy values in three batches to manage gas costs and allows only a specified caller to retrieve these values. The contract includes functions to set and get the allowed caller, retrieve the next entropy value, and derive various parameters based on entropy. It also has safety checks and utility functions for entropy calculations and digit manipulation. The contract uses OpenZeppelin's Ownable and Pausable for access control and emergency stops.

### Questions 
- Is the onlyAllowedCaller modifier secure and correctly implemented?
Are there any potential reentrancy issues?

- How secure and unpredictable is the pseudo-randomness generated by the contract?
Are there any potential biases or weaknesses in the entropy generation method?

- Are the batch initialization functions (writeEntropyBatch1, writeEntropyBatch2, writeEntropyBatch3) optimized for gas usage?
Is there a risk of running out of gas in any of the loops?

- Are there comprehensive unit tests covering all functions and edge cases?

- Are the imported OpenZeppelin contracts up-to-date and secure?
Is the interface IEntropyGenerator correctly implemented and used?

- Is the contract designed with upgradeability in mind?


### suggested Improvements 

-Batch Initialization: Optimize loops in writeEntropyBatch1, writeEntropyBatch2, and writeEntropyBatch3 to minimize gas usage.
Consider using a more efficient data structure for entropy storage.

- Provide more descriptive error messages in require statements.

- Add more detailed comments explaining complex logic and calculations.

- Proxy Pattern: Consider using the proxy pattern for upgradeability if future changes are anticipated.

## Low and Non-critical Vulnerabilities

### [NC-01] Use Chainlink VRF or another secure randomness oracle instead of block data for better unpredictability.
