

### Transferred `ERC721` can be stuck permanently
If the recipient is not a EOA, safeTransferFrom ensures that the contract is able to safely receive the token. In the worst-case scenario, it may result in tokens frozen permanently, as the following code uses `transferFrom`, which [doesn't check](https://github.com/ethereum/EIPs/blob/78e2c297611f5e92b6a5112819ab71f74041ff25/EIPS/eip-721.md?plain=1#L103-L113) if the recipient can handle the NFT.

```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

53:    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract	// @audit-issue

81:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer NFT to the buyer	// @audit-issue

104:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer the nft back to seller	// @audit-issue
```
[53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L53-L53), [81](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L81-L81), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L104-L104), 


#### Recommendation

To prevent `ERC721` tokens from being permanently stuck when transferring to non-EOA addresses, consider using the `safeTransferFrom` function instead of `transferFrom`. The `safeTransferFrom` function includes additional checks to ensure that the recipient can handle the NFT, reducing the risk of tokens getting stuck in contracts that cannot manage them. This extra precaution helps maintain the usability and functionality of your `ERC721` token transfers.

### `block.number` means different things on different L2s
On Optimism, `block.number` is the L2 block number, but on Arbitrum, it's the L1 block number, and `ArbSys(address(100)).arbBlockNumber()` must be used. Furthermore, L2 block numbers often occur much more frequently than L1 block numbers (any may even occur on a per-transaction basis), so using block numbers for timing results in inconsistencies, especially when voting is involved across multiple chains. As of version 4.9, OpenZeppelin has [modified](https://blog.openzeppelin.com/introducing-openzeppelin-contracts-v4.9#governor) their governor code to use a clock rather than block numbers, to avoid these sorts of issues, but this still requires that the project [implement](https://docs.openzeppelin.com/contracts/4.x/governance#token_2) a [clock](https://eips.ethereum.org/EIPS/eip-6372) for each L2.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

54:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

74:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

92:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

208:      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L54-L54), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L74-L74), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L92-L92), [208](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L208-L208), 


#### Recommendation

Adopt a consistent timekeeping mechanism across L1 and L2 solutions when using `block.number` or similar timing mechanisms in your Solidity contracts. Consider using a standard clock or timestamp-based system, especially for functionalities like governance that require consistent timing across chains. For contracts on specific L2 solutions, ensure you're using the correct method to obtain block numbers consistent with the intended behavior. Keep abreast of standards and best practices, such as those suggested by OpenZeppelin, and implement appropriate solutions like EIP-6372 to provide a consistent and reliable timing mechanism across different blockchain environments.

### Consider implementing two-step procedure for updating protocol addresses
Implementing a two-step procedure for updating protocol addresses adds an extra layer of security. In such a system, the first step initiates the change, and the second step, after a predefined delay, confirms and finalizes it. This delay allows stakeholders or monitoring tools to observe and react to unintended or malicious changes. If an unauthorized change is detected, corrective actions can be taken before the change is finalized. To achieve this, introduce a "proposed address" state variable and a "delay period". Upon an update request, set the "proposed address". After the delay, if not contested, the main protocol address can be updated.

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

33:    nukeFundAddress = _nukeFundAddress;	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L33-L33), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

30:    nukeFundAddress = _nukeFundAddress;	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L30-L30), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

85:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

90:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

95:    devAddress = account;	// @audit-issue

100:    daoAddress = account;	// @audit-issue
```
[85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L85-L85), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L90-L90), [95](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L95-L95), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L100-L100), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

37:    allowedCaller = _allowedCaller;	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L37-L37), 


#### Recommendation

Introduce two state variables in your contract: one to hold the proposed new address (`address public proposedAddress`) and another to timestamp the proposal (`uint public addressChangeInitiated`). Implement two functions: one to propose a new address, recording the current timestamp, and another to finalize the address change after the delay period has elapsed.

### Centralization risk for privileged functions
Contracts with privileged functions need owner/admin to be trusted not to perform malicious updates or drain funds. This may also cause a single point failure.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

66:  function setNukeFundContract(
67:    address payable _nukeFundAddress
68:  ) external onlyOwner {	// @audit-issue

74:  function setEntityForgingContract(
75:    address entityForgingAddress_
76:  ) external onlyOwner {	// @audit-issue

82:  function setEntropyGenerator(
83:    address entropyGeneratorAddress_
84:  ) external onlyOwner {	// @audit-issue

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue

96:  function startAirdrop(uint256 amount) external whenNotPaused onlyOwner {	// @audit-issue

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {	// @audit-issue

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {	// @audit-issue

108:  function setPriceIncrementByGen(
109:    uint256 _priceIncrementByGen
110:  ) external onlyOwner {	// @audit-issue

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue

181:  function mintToken(
182:    bytes32[] calldata proof
183:  )
184:    public
185:    payable
186:    whenNotPaused
187:    nonReentrant
188:    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))	// @audit-issue

202:  function mintWithBudget(
203:    bytes32[] calldata proof
204:  )
205:    public
206:    payable
207:    whenNotPaused
208:    nonReentrant
209:    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))	// @audit-issue
```
[68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L66-L68), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L74-L76), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L82-L84), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L96-L96), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L100), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L104), [110](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L110), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L188), [209](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L209), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

30:  function setNukeFundAddress(
31:    address payable _nukeFundAddress
32:  ) external onlyOwner {	// @audit-issue

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue

40:  function setOneYearInDays(uint256 value) external onlyOwner {	// @audit-issue

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {	// @audit-issue
```
[32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L32), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L36), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L44), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

30:  function addDev(address user, uint256 weight) external onlyOwner {	// @audit-issue

40:  function updateDev(address user, uint256 weight) external onlyOwner {	// @audit-issue

51:  function removeDev(address user) external onlyOwner {	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L30-L30), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L40-L40), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L51-L51), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

27:  function setNukeFundAddress(
28:    address payable _nukeFundAddress
29:  ) external onlyOwner {	// @audit-issue

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue
```
[29](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L29), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L33), 


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

To mitigate centralization risks associated with privileged functions, consider implementing a multi-signature or decentralized governance mechanism. Instead of relying solely on a single owner/administrator, distribute control and decision-making authority among multiple parties or stakeholders. This approach enhances security, reduces the risk of malicious actions by a single entity, and prevents single points of failure. Explore governance solutions and smart contract frameworks that support decentralized control and decision-making to enhance the trustworthiness and resilience of your contract.

### Code does not follow the best practice of check-effects-interactions pattern
Code should follow the best-practice of [CEI](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

53:    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract
54:
55:    ++listingCount;	// @audit-issue

53:    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract
54:
55:    ++listingCount;
56:    listings[listingCount] = Listing(msg.sender, tokenId, price, true);	// @audit-issue

53:    nftContract.transferFrom(msg.sender, address(this), tokenId); // trasnfer NFT to contract
54:
55:    ++listingCount;
56:    listings[listingCount] = Listing(msg.sender, tokenId, price, true);
57:    listedTokenIds[tokenId] = listingCount;	// @audit-issue

77:    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(
78:      ''
79:    );
80:    require(success, 'Failed to send to seller');
81:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer NFT to the buyer
82:
83:    delete listings[listedTokenIds[tokenId]]; // remove listing	// @audit-issue

104:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer the nft back to seller
105:
106:    delete listings[listedTokenIds[tokenId]]; // mark the listing as inactive or delete it	// @audit-issue
```
[55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L53-L55), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L53-L56), [57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L53-L57), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L77-L83), [106](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L104-L106), 


#### Recommendation

To enhance the security of your smart contract and prevent reentrancy attacks, it's crucial to adhere to the check-effects-interactions pattern. Restructure your contract's functions to follow this pattern by first performing condition checks (checks), then updating the contract's state (effects), and finally, executing any external calls or interactions (interactions). This sequential approach reduces the risk of reentrancy vulnerabilities by ensuring that state changes are isolated from external interactions and condition checks are made before any state modifications. Review and refactor your contract code to align with this best practice for improved security.

### Missing Reentrancy Modifier
A reentrancy attack is a common vulnerability in smart contracts, particularly when a contract makes an external call to another untrusted contract and then continues to execute code afterwards. If the called contract is malicious, it can call back into the original contract in a way that causes the original function to run again, potentially leading to effects like multiple withdrawals and other unintended actions.

The absence of reentrancy guards, such as the `nonReentrant` modifier provided by OpenZeppelin's ReentrancyGuard contract, makes a function susceptible to reentrancy attacks. This modifier prevents a function from being called again until it has finished executing.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

361:    (bool success, ) = nukeFundAddress.call{ value: totalAmount }('');	// @audit-issue
```
[361](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L361-L361), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

20:        (bool success, ) = payable(owner()).call{ value: remaining }('');	// @audit-issue

24:      (bool success, ) = payable(owner()).call{ value: msg.value }('');	// @audit-issue
```
[20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L20-L20), [24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L24-L24), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

114:    (bool success, ) = nukeFundAddress.call{ value: amount }('');	// @audit-issue
```
[114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L114-L114), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

47:      (bool success, ) = devAddress.call{ value: devShare }('');	// @audit-issue

51:      (bool success, ) = payable(owner()).call{ value: devShare }('');	// @audit-issue

54:      (bool success, ) = daoAddress.call{ value: devShare }('');	// @audit-issue
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L47), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L51-L51), [54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L54-L54), 


#### Recommendation

Consider adding `nonReentrant` modifier to given functions.

### Missing checks for `address(0)` in constructor/initializers
In Solidity, the Ethereum address `0x0000000000000000000000000000000000000000` is known as the "zero address". This address has significance because it's the default value for uninitialized address variables and is often used to represent an invalid or non-existent address. The "
Missing zero address control" issue arises when a Solidity smart contract does not properly check or prevent interactions with the zero address, leading to unintended behavior.
For instance, a contract might allow tokens to be sent to the zero address without any checks, which essentially burns those tokens as they become irretrievable. While sometimes this is intentional, without proper control or checks, accidental transfers could occur.    
        

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

26:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L26-L26), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

23:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue
```
[23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L23-L23), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

33:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

34:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

35:    devAddress = _devAddress; // Set the developer's address	// @audit-issue

36:    daoAddress = _daoAddress;	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L33-L33), [34](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L34-L34), [35](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L35-L35), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L36-L36), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

31:    allowedCaller = _traitForgetNft;	// @audit-issue
```
[31](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L31-L31), 


#### Recommendation

It is strongly recommended to implement checks to prevent the zero address from being set during the initialization of contracts. This can be achieved by adding require statements that ensure address parameters are not the zero address. 

### Missing checks for `address(0)`  when updating state variables
In Solidity, the Ethereum address `0x0000000000000000000000000000000000000000` is known as the "zero address". This address has significance because it's the default value for uninitialized address variables and is often used to represent an invalid or non-existent address. The "
Missing zero address control" issue arises when a Solidity smart contract does not properly check or prevent interactions with the zero address, leading to unintended behavior.
For instance, a contract might allow tokens to be sent to the zero address without any checks, which essentially burns those tokens as they become irretrievable. While sometimes this is intentional, without proper control or checks, accidental transfers could occur.    
        

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

69:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

294:    initialOwners[newItemId] = to;	// @audit-issue

329:    initialOwners[newTokenId] = newOwner;	// @audit-issue
```
[69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L69-L69), [294](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L294-L294), [329](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L329-L329), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

33:    nukeFundAddress = _nukeFundAddress;	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L33-L33), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

30:    nukeFundAddress = _nukeFundAddress;	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L30-L30), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

85:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

90:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

95:    devAddress = account;	// @audit-issue

100:    daoAddress = account;	// @audit-issue
```
[85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L85-L85), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L90-L90), [95](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L95-L95), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L100-L100), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

37:    allowedCaller = _allowedCaller;	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L37-L37), 


#### Recommendation

It is strongly recommended to implement checks to prevent the zero address from being set during the initialization of contracts. This can be achieved by adding require statements that ensure address parameters are not the zero address. 

### Do not allow fees to be set to `100%`
It is recommended from a risk perspective to disallow setting 100% fees at all. A reasonable fee maximum should be checked for instead.

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

45:    minimumListFee = _fee;	// @audit-issue
```
[45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L45-L45), 


#### Recommendation

Implement strict validation checks in your smart contract's fee-setting functions to ensure that fees cannot be configured to 100%. Define a maximum allowable fee percentage that preserves the contract's functionality and user experience, such as 10% or 20%, depending on your project's specific requirements and economic model. For example:
```solidity
function setTransactionFee(uint256 newFeePercentage) public onlyOwner {
    require(newFeePercentage <= 20, "Fee cannot exceed 20%");
    transactionFee = newFeePercentage;
}
```


### Gas griefing/theft is possible on an unsafe external call
A low-level call will copy any amount of bytes to local memory. When bytes are copied from returndata to memory, the memory expansion cost is paid.

This means that when using a standard solidity call, the callee can 'returnbomb' the caller, imposing an arbitrary gas cost.

Because this gas is paid by the caller and in the caller's context, it can cause the caller to run out of gas and halt execution.

Consider replacing all unsafe `call` with `excessivelySafeCall` from this [repository](https://github.com/nomad-xyz/ExcessivelySafeCall).


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

158:    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');	// @audit-issue
```
[158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L158-L158), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

89:    (bool success, ) = payable(to).call{ value: amount }('');	// @audit-issue
```
[89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L89-L89), 


#### Recommendation

Mitigate the risk of gas griefing by replacing direct low-level calls (`call`, `delegatecall`, etc.) with `excessivelySafeCall`, a utility that limits the amount of gas used for copying return data, thereby preventing return bombing. The `excessivelySafeCall` function, available in certain security-focused repositories such as [nomad-xyz/ExcessivelySafeCall](https://github.com/nomad-xyz/ExcessivelySafeCall), implements additional checks to ensure that the callee cannot impose arbitrary gas costs on the caller. To incorporate this into your contract:

1. Review your contract for external calls to untrusted contracts.
2. Import or implement an `excessivelySafeCall` utility in your contract.
3. Replace unsafe external calls with `excessivelySafeCall`, specifying limits on return data size as appropriate for your application.

Example usage:
```solidity
// Assuming excessivelySafeCall is available in your contract
(bool success, ) = target.excessivelySafeCall(gasLimit, callData, maxReturnSize);
require(success, "External call failed");

This approach not only protects against gas griefing attacks but also enhances the overall security posture of your contract by ensuring that external interactions do not lead to unexpected execution halts or excessive gas consumption. Always perform thorough testing after integrating `excessivelySafeCall` to ensure compatibility and intended functionality.

### `_safeMint()` should be used rather than `_mint()` wherever possible
`_mint()` is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of `_safeMint()` which ensures that the recipient is either an EOA or implements `IERC721Receiver`. Both [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function. In the cases below, `_mint()` does not call `ERC721TokenReceiver.onERC721Received()` on the recipient.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

287:    _mint(to, newItemId);	// @audit-issue

323:    _mint(newOwner, newTokenId);	// @audit-issue
```
[287](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L287-L287), [323](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L323-L323), 


#### Recommendation

Update your ERC721 token contracts to use `_safeMint()` for minting new tokens, replacing any instances of `_mint()`. This ensures that the minting process includes a check for the recipient's ability to handle the token, safeguarding against the risk of token loss. Ensure that your contract, or any contracts you interact with, correctly implement the `IERC721Receiver` interface when receiving tokens. Review the documentation and implementation examples provided by OpenZeppelin and solmate to understand the usage and benefits of `_safeMint()`. Incorporating `_safeMint()` enhances your contract's security and compatibility with the ERC721 standard, making your token safer and more reliable for users and developers alike.

### Use `Ownable2Step` rather than `Ownable`
[Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3d7a93876a2e5e1d7fe29b5a0e96e222afdc4cfa/contracts/access/Ownable2Step.sol#L31-L56) and [Ownable2StepUpgradeable](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/access/Ownable2StepUpgradeable.sol#L47-L63) prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

18:  Ownable,	// @audit-issue
```
[18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L18-L18), 


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

Consider using `Ownable2Step` or `Ownable2StepUpgradeable` from OpenZeppelin Contracts to enhance the security of your contract ownership management. These contracts prevent the accidental transfer of ownership to an address that cannot handle it, such as due to a typo, by requiring the recipient of owner permissions to actively accept ownership via a contract call. This two-step ownership transfer process adds an additional layer of security to your contract's ownership management.

### Consider using OpenZeppelin’s `SafeCast` library to prevent unexpected overflows when casting from various type int/uint values
In Solidity, when casting from `int` to `uint` or vice versa, there is a risk of unexpected overflow or underflow, especially when dealing with large values. To mitigate this risk and ensure safe type conversions, you can consider using OpenZeppelin’s `SafeCast` library. This library provides functions that check for overflow/underflow conditions before performing the cast, helping you prevent potential issues related to type conversion in your smart contracts.

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

86:    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue

138:    uint8 mergerForgePotential = uint8((mergerEntropy / 10) % 10); // Extract the 5th digit from the entropy	// @audit-issue
```
[86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L86-L86), [138](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L138-L138), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

53:        uint256 pseudoRandomValue = uint256(	// @audit-issue

55:        ) % uint256(10) ** 78; // generate a  pseudo-random value using block number and index	// @audit-issue

73:        uint256 pseudoRandomValue = uint256(	// @audit-issue

75:        ) % uint256(10) ** 78;	// @audit-issue

91:        uint256 pseudoRandomValue = uint256(	// @audit-issue

93:        ) % uint256(10) ** 78;	// @audit-issue

207:    uint256 hashValue = uint256(	// @audit-issue
```
[53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L53-L53), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L55-L55), [73](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L73-L73), [75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L75-L75), [91](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L91-L91), [93](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L93-L93), [207](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L207-L207), 


#### Recommendation

To enhance the safety and reliability of your Solidity smart contracts, it is advisable to utilize OpenZeppelin’s `SafeCast` library when casting between `int` and `uint` types. Incorporating this library into your codebase will help prevent unexpected overflows and underflows during type conversion, reducing the risk of vulnerabilities and ensuring secure contract execution.

### State variables not limited to reasonable values
Consider adding appropriate minimum/maximum value checks to ensure that the following state variables can never be used to excessively harm users, including via griefing.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

101:    startPrice = _startPrice;	// @audit-issue

105:    priceIncrement = _priceIncrement;	// @audit-issue

111:    priceIncrementByGen = _priceIncrementByGen;	// @audit-issue

127:    whitelistEndTime = endTime_;	// @audit-issue

326:    tokenEntropy[newTokenId] = entropy;	// @audit-issue

327:    tokenGenerations[newTokenId] = gen;	// @audit-issue
```
[101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L101-L101), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L105-L105), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L111-L111), [127](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L127-L127), [326](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L326-L326), [327](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L327-L327), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

37:    taxCut = _taxCut;	// @audit-issue

41:    oneYearInDays = value;	// @audit-issue

96:    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L37-L37), [41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L41-L41), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L96-L96), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

34:    taxCut = _taxCut;	// @audit-issue

56:    listings[listingCount] = Listing(msg.sender, tokenId, price, true);	// @audit-issue
```
[34](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L34-L34), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L56-L56), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

64:    taxCut = _taxCut;	// @audit-issue

68:    minimumDaysHeld = value;	// @audit-issue

72:    defaultNukeFactorIncrease = value;	// @audit-issue

76:    maxAllowedClaimDivisor = value;	// @audit-issue

80:    nukeFactorMaxParam = value;	// @audit-issue

110:    ageMultiplier = _ageMultiplier;	// @audit-issue
```
[64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L64-L64), [68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L68-L68), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L72-L72), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L76-L76), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L80-L80), [110](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L110-L110), 


#### Recommendation


Implement validation checks for state variables to enforce minimum and maximum value limits. This can be achieved by adding modifiers or require statements in Solidity functions that modify these state variables. Ensure that these limits are reasonable and reflect the intended use of the contract. Additionally, consider implementing a mechanism to update these limits through a governance process or a trusted role, if applicable, to maintain flexibility and adaptability of the contract over time.

### Missing contract existence checks before low-level calls
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`.

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
Path: ./contracts/NukeFund/NukeFund.sol

47:      (bool success, ) = devAddress.call{ value: devShare }('');	// @audit-issue

51:      (bool success, ) = payable(owner()).call{ value: devShare }('');	// @audit-issue

54:      (bool success, ) = daoAddress.call{ value: devShare }('');	// @audit-issue

177:    (bool success, ) = payable(msg.sender).call{ value: claimAmount }('');	// @audit-issue
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L47), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L51-L51), [54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L54-L54), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L177-L177), 


#### Recommendation

To enhance the security and reliability of your Solidity smart contracts, always include contract existence checks before making low-level calls. In addition to verifying that the address is not the zero address, also confirm that `<address>.code.length > 0`. These checks help ensure that the target address corresponds to a valid and functioning contract, reducing the risk of unexpected behavior and vulnerabilities in your contract interactions.

### Solidity version 0.8.20 might not work on all chains due to `PUSH0`
The Solidity version 0.8.20 employs the recently introduced PUSH0 opcode in the Shanghai EVM. This opcode might not be universally supported across all blockchain networks and Layer 2 solutions. Thus, as a result, it might be not possible to deploy solution with version 0.8.20 >= on some blockchains.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L2-L2), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L2-L2), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L2-L2), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L2-L2), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L2-L2), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L2-L2), 


#### Recommendation

The Solidity version 0.8.20 employs the recently introduced PUSH0 opcode in the Shanghai EVM. This opcode might not be universally supported across all blockchain networks and Layer 2 solutions. Thus, as a result, it might be not possible to deploy solution with version 0.8.20 >= on some blockchains.

It is recommended to verify whether solution can be deployed on particular blockchain with the Solidity version 0.8.20 >=. Whenever such deployment is not possible due to lack of PUSH0 opcode support and lowering the Solidity version is a must, it is strongly advised to review all feature changes and bugfixes in [Solidity releases](https://soliditylang.org/blog/category/releases/). Some changes may have impact on current implementation and may impose a necessity of maintaining another version of solution.

### `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`
Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). "Unless there is a compelling reason, `abi.encode` should be preferred". If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).
If all arguments are strings and or bytes, `bytes.concat()` should be used instead

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

54:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

74:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

92:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

208:      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L54-L54), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L74-L74), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L92-L92), [208](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L208-L208), 


#### Recommendation

To ensure the integrity of hash functions like `keccak256()` in Solidity, prefer using `abi.encode()` instead of `abi.encodePacked()` when dealing with dynamic types. `abi.encode()` pads items to 32 bytes, preventing hash collisions and enhancing security. Only use `abi.encodePacked()` when there is a compelling reason. If there's only one argument to `abi.encodePacked()`, consider casting it to `bytes()` or `bytes32()`. If all arguments are strings or bytes, use `bytes.concat()` for better clarity and reliability.

### Critical functions should have a timelock
Critical functions, especially those affecting protocol parameters or user funds, are potential points of failure or exploitation. To mitigate risks, incorporating a timelock on such functions can be beneficial. A timelock requires a waiting period between the time an action is initiated and when it's executed, giving stakeholders time to react, potentially vetoing malicious or erroneous changes. To implement, integrate a smart contract like OpenZeppelin's `TimelockController` or build a custom mechanism. This ensures governance decisions or administrative changes are transparent and allows for community or multi-signature interventions, enhancing protocol security and trustworthiness.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

66:  function setNukeFundContract(
67:    address payable _nukeFundAddress
68:  ) external onlyOwner {
69:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {
101:    startPrice = _startPrice;	// @audit-issue

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {
105:    priceIncrement = _priceIncrement;	// @audit-issue

108:  function setPriceIncrementByGen(
109:    uint256 _priceIncrementByGen
110:  ) external onlyOwner {
111:    priceIncrementByGen = _priceIncrementByGen;	// @audit-issue

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {
123:    rootHash = rootHash_;	// @audit-issue

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {
127:    whitelistEndTime = endTime_;	// @audit-issue
```
[69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L66-L69), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L101), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L105), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L111), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L123), [127](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L127), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

30:  function setNukeFundAddress(
31:    address payable _nukeFundAddress
32:  ) external onlyOwner {
33:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {
37:    taxCut = _taxCut;	// @audit-issue

40:  function setOneYearInDays(uint256 value) external onlyOwner {
41:    oneYearInDays = value;	// @audit-issue

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {
45:    minimumListFee = _fee;	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L33), [37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L37), [41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L41), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L45), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

27:  function setNukeFundAddress(
28:    address payable _nukeFundAddress
29:  ) external onlyOwner {
30:    nukeFundAddress = _nukeFundAddress;	// @audit-issue

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {
34:    taxCut = _taxCut;	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L30), [34](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L34), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

63:  function setTaxCut(uint256 _taxCut) external onlyOwner {
64:    taxCut = _taxCut;	// @audit-issue

67:  function setMinimumDaysHeld(uint256 value) external onlyOwner {
68:    minimumDaysHeld = value;	// @audit-issue

71:  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {
72:    defaultNukeFactorIncrease = value;	// @audit-issue

75:  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {
76:    maxAllowedClaimDivisor = value;	// @audit-issue

79:  function setNukeFactorMaxParam(uint256 value) external onlyOwner {
80:    nukeFactorMaxParam = value;	// @audit-issue

84:  function setTraitForgeNftContract(address _traitForgeNft) external onlyOwner {
85:    nftContract = ITraitForgeNft(_traitForgeNft);	// @audit-issue

89:  function setAirdropContract(address _airdrop) external onlyOwner {
90:    airdropContract = IAirdrop(_airdrop);	// @audit-issue

94:  function setDevAddress(address payable account) external onlyOwner {
95:    devAddress = account;	// @audit-issue

99:  function setDaoAddress(address payable account) external onlyOwner {
100:    daoAddress = account;	// @audit-issue

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {
110:    ageMultiplier = _ageMultiplier;	// @audit-issue
```
[64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L64), [68](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L67-L68), [72](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L71-L72), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L75-L76), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L79-L80), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L84-L85), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L89-L90), [95](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L94-L95), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L99-L100), [110](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L110), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

36:  function setAllowedCaller(address _allowedCaller) external onlyOwner {
37:    allowedCaller = _allowedCaller;	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L36-L37), 


#### Recommendation

Integrate a timelock mechanism into your Solidity contracts for critical functions, especially those controlling protocol parameters or managing user funds. Consider using established solutions like OpenZeppelin's `TimelockController` for robustness and reliability. Alternatively, develop a custom timelock mechanism tailored to your specific requirements. Ensure that the timelock duration is appropriate for your contract's use case and stakeholder needs, providing sufficient time for review and intervention. Clearly document and communicate the presence and workings of the timelock mechanism to stakeholders to maintain transparency and trust in your contract's operations.

### Possible loss of precision
Division by large numbers may result in precision loss due to rounding down, or even the result being erroneously equal to zero. Consider adding checks on the numerator to ensure precision loss is handled appropriately.


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

146:    uint256 devFee = forgingFee / taxCut;	// @audit-issue
```
[146](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L146-L146), 


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

147:    uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) /	// @audit-issue
148:      MAX_DENOMINATOR) + initialNukeFactor;

166:    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; // Calculate the potential claim amount based on the finalNukeFactor	// @audit-issue

167:    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size	// @audit-issue
```
[41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L41-L41), [147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L147-L148), [166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L166-L166), [167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L167-L167), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue
```
[181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), 


#### Recommendation

Incorporate strategies in your Solidity contracts to mitigate precision loss in division operations. This can include:
1. Performing checks on the numerator and denominator to ensure they are within a reasonable range to avoid significant rounding errors.
2. Considering the use of fixed-point arithmetic libraries or scaling factors to handle divisions with higher precision.
3. Clearly documenting any inherent limitations of your division logic and providing guidelines for inputs to minimize unexpected behavior.
Always thoroughly test division operations under various scenarios to ensure that the outcomes are consistent with your contract's intended logic and accuracy requirements.

### A year is not always 365 days
On leap years, the number of days is 366, so calculations during those years will return the wrong value

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

14:  uint256 public oneYearInDays = 365 days;	// @audit-issue
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L14-L14), 


#### Recommendation

Incorporate leap year recognition into your smart contract's time-based calculations to ensure accuracy across all years. Utilize a function that calculates the exact number of days in a given year by checking for leap years. For example, a year is a leap year if it is divisible by 4 but not by 100, unless it is also divisible by 400.


### Floating Pragma
A "floating pragma" in Solidity refers to the practice of using a pragma statement that does not specify a fixed compiler version but instead allows the contract to be compiled with any compatible compiler version. This issue arises when pragma statements like `pragma solidity ^0.8.0;` are used without a specific version number, allowing the contract to be compiled with the latest available compiler version. This can lead to various compatibility and stability issues.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L2-L2), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L2-L2), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L2-L2), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L2-L2), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L2-L2), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L2-L2), 


#### Recommendation

Consider locking the pragma version whenever possible and avoid using a floating pragma in the final deployment. [Consider known bugs](https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### Ownership Irrevocability Vulnerability
The smart contract under inspection inherits from the `Ownable` library, which provides basic authorization control functions, simplifying the implementation of user permissions. However, the contract does not provide a mechanism to transfer ownership to another address or account, and it retains the default `renounceOwnership` function from `Ownable`.  
Given this, once the owner renounces ownership using the `renounceOwnership` function, the contract becomes ownerless. As evidenced in the provided transaction logs, after the `renounceOwnership` function is called, attempts to call functions that require owner permissions fail with the error message: `"Ownable: caller is not the owner."`  
This state renders the contract's adjustable parameters immutable and potentially makes the contract useless for any future administrative changes that might be necessary.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

18:  Ownable,	// @audit-issue
```
[18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L18-L18), 


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


To mitigate this vulnerability:
* Override the renounceOwnership function to revert transactions: By overriding this function to simply revert any transaction, it will become impossible for the contract owner to unintentionally (or intentionally) render the contract ownerless and thus immutable.
* Implement an ownership transfer function: While the Ownable library does provide a transferOwnership function, if this is not present or has been removed from the current contract, it should be re-implemented to ensure there is a way to transfer ownership in future scenarios.


### Unneeded initializations of integer variable to `0`.
In Solidity, it is common practice to initialize variables with default values when declaring them. However, initializing integer variables to `0` when they are not subsequently used in the code can lead to unnecessary gas consumption and code clutter. This issue points out instances where such initializations are present but serve no functional purpose.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

212:    uint256 amountMinted = 0;	// @audit-issue
```
[212](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L212-L212), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

15:  uint256 public listingCount = 0;	// @audit-issue
```
[15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L15-L15), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

16:  uint256 public listingCount = 0;	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L16-L16), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

11:  uint256 private lastInitializedIndex = 0; // Indexes to keep track of the initialization and usage of entropy values	// @audit-issue

12:  uint256 private currentSlotIndex = 0;	// @audit-issue

13:  uint256 private currentNumberIndex = 0;	// @audit-issue

189:    uint256 digits = 0;	// @audit-issue
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L12-L12), [13](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L13-L13), [189](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L189-L189), 


#### Recommendation


It is recommended not to initialize integer variables to `0` to save some Gas.


### Use transfer libraries instead of low level calls
Consider using `SafeTransferLib.safeTransferETH` or `Address.sendValue` for clearer semantic meaning instead of using a low level call.

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

To improve code readability and safety, consider using higher-level transfer libraries like `SafeTransferLib.safeTransferETH` or `Address.sendValue` instead of low-level calls for handling Ether transfers. These libraries provide clearer semantic meaning and help prevent common pitfalls associated with low-level calls.

### Unused arguments should be removed or implemented
In Solidity, functions often have arguments that are intended for specific operations or logic within the function. However, sometimes these arguments remain unused in the function body, either due to changes during development or oversight. Unused arguments in functions can lead to confusion, making the code less readable and potentially misleading for other developers who might use or audit the contract. Moreover, they can create a false impression of the function's purpose and behavior. It's crucial to either implement these arguments in the function's logic as originally intended or remove them to ensure clarity and efficiency of the code.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

157:    string memory	// @audit-issue: None
```
[157](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L157-L157), 


#### Recommendation

Eliminate unused arguments in Solidity function definitions to enhance code clarity and efficiency. If an argument is currently not utilized in the function's logic, assess its potential future utility. If it serves no purpose, removing it simplifies the function signature and aligns the code more closely with its actual operation, contributing to a cleaner and more understandable contract structure.

### The `nonReentrant modifier` should occur before all other modifiers
In Solidity, the order in which modifiers are applied to a function can significantly impact the function's behavior and security. The `nonReentrant` modifier is crucial for preventing reentrancy attacks, a common vulnerability in smart contracts. This modifier should be the first in the list of applied modifiers to ensure that the reentrancy check is performed before any other logic in the function. Placing `nonReentrant` after other modifiers could potentially lead to security weaknesses, as other modifiers might perform state changes or external calls before the reentrancy protection takes effect. Therefore, correct positioning of the `nonReentrant` modifier is essential for maintaining the integrity and security of the function and, by extension, the entire contract.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

141:  function burn(uint256 tokenId) external whenNotPaused nonReentrant {	// @audit-issue

153:  function forge(
154:    address newOwner,
155:    uint256 parent1Id,
156:    uint256 parent2Id,
157:    string memory
158:  ) external whenNotPaused nonReentrant returns (uint256) {	// @audit-issue

181:  function mintToken(
182:    bytes32[] calldata proof
183:  )

202:  function mintWithBudget(
203:    bytes32[] calldata proof
204:  )
```
[141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L141-L141), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L158), [187](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L183), [208](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L204), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

67:  function listForForging(
68:    uint256 tokenId,
69:    uint256 fee
70:  ) public whenNotPaused nonReentrant {	// @audit-issue

102:  function forgeWithListed(
103:    uint256 forgerTokenId,
104:    uint256 mergerTokenId
105:  ) external payable whenNotPaused nonReentrant returns (uint256) {	// @audit-issue

177:  function cancelListingForForging(
178:    uint256 tokenId
179:  ) external whenNotPaused nonReentrant {	// @audit-issue
```
[70](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L67-L70), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L105), [179](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L177-L179), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

61:  function claim() external whenNotPaused nonReentrant {	// @audit-issue
```
[61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L61-L61), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

38:  function listNFTForSale(
39:    uint256 tokenId,
40:    uint256 price
41:  ) public whenNotPaused nonReentrant {	// @audit-issue

63:  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {	// @audit-issue

94:  function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue
```
[41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L38-L41), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L63-L63), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L94-L94), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

153:  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue
```
[153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L153-L153), 


#### Recommendation

To ensure the effectiveness of the `nonReentrant` modifier in protecting against reentrancy, it should be placed as the first modifier in a function's list of modifiers, before all other modifiers. This helps prevent reentrancy attacks from being triggered by other modifiers.

### Payable Usage Is Unnecessary
The use of the `payable` keyword to convert addresses before sending Ether in Solidity can sometimes be redundant, particularly when the target address (`to`) could be defined as `payable` in the function parameters. This redundancy not only adds unnecessary complexity to the code but also obscures the function's intention of transferring Ether to a specific address. Defining the address as `payable` from the outset clarifies that the function is intended to perform Ether transfers and ensures that the address type is correctly specified for such transactions.

```solidity
Path: ./contracts/DevFund/DevFund.sol

89:    (bool success, ) = payable(to).call{ value: amount }('');	// @audit-issue: variable `to` could be defined in the function parameters as `payable to` instead of using `payable` function.
```
[89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L89-L89), 


#### Recommendation

Review your contract's functions to identify instances where the `payable` keyword is used to convert addresses just before making a call to transfer Ether. Refactor these functions by specifying the `payable` keyword in the function parameters for addresses intended to receive Ether. This practice enhances code clarity, reduces unnecessary conversions, and explicitly indicates which addresses are expected to participate in Ether transactions.

Example before refactoring:
```solidity
function transferEther(address to, uint256 amount) external {
    (bool success, ) = payable(to).call{value: amount}("");
    require(success, "Transfer failed.");
}
```

Example after refactoring:
```solidity
function transferEther(address payable to, uint256 amount) external {
    (bool success, ) = to.call{value: amount}("");
    require(success, "Transfer failed.");
}
```

This approach streamlines the function's design and makes the contract's intentions regarding Ether transfers more transparent.


### Excessive Authorization in Contract Functions
A contract where a high percentage of functions require authorization (e.g., restricted to the contract owner or specific roles) may indicate over-centralization or excessive control. This could limit the contract's flexibility, reduce trust among users, and potentially create bottleneck points that could be exploited or become failure points. While some level of control is necessary for administrative purposes, overly restrictive access can detract from the decentralized nature of blockchain applications and concentrate too much power in the hands of a few.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue: %93.33333333333333 amount of external/public and non-view/non-pure functions are required authorization to call.
	List of total functions: `setPriceIncrementByGen`, `mintToken`, `setEntropyGenerator`, `setEntityForgingContract`, `setStartPrice`, `startAirdrop`, `setMaxGeneration`, `setWhitelistEndTime`, `mintWithBudget`, `setRootHash`, `burn`, `forge`, `setNukeFundContract`, `setAirdropContract`, `setPriceIncrement`
	List of functions that require authorization: `setPriceIncrementByGen`, `mintToken`, `setEntropyGenerator`, `setEntityForgingContract`, `setStartPrice`, `startAirdrop`, `setMaxGeneration`, `setWhitelistEndTime`, `mintWithBudget`, `setRootHash`, `forge`, `setNukeFundContract`, `setAirdropContract`, `setPriceIncrement`
	List of functions that doesn't require authorization: `burn`
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L14), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

9:contract DevFund is IDevFund, Ownable, ReentrancyGuard, Pausable {	// @audit-issue: %75.0 amount of external/public and non-view/non-pure functions are required authorization to call.
	List of total functions: `removeDev`, `updateDev`, `addDev`, `claim`
	List of functions that require authorization: `removeDev`, `updateDev`, `addDev`
	List of functions that doesn't require authorization: `claim`
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L9-L9), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

11:contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {	// @audit-issue: %90.9090909090909 amount of external/public and non-view/non-pure functions are required authorization to call.
	List of total functions: `setDevAddress`, `setAgeMultplier`, `setDefaultNukeFactorIncrease`, `setMaxAllowedClaimDivisor`, `nuke`, `setNukeFactorMaxParam`, `setTaxCut`, `setTraitForgeNftContract`, `setAirdropContract`, `setDaoAddress`, `setMinimumDaysHeld`
	List of functions that require authorization: `setDevAddress`, `setAgeMultplier`, `setDefaultNukeFactorIncrease`, `setMaxAllowedClaimDivisor`, `setNukeFactorMaxParam`, `setTaxCut`, `setTraitForgeNftContract`, `setAirdropContract`, `setDaoAddress`, `setMinimumDaysHeld`
	List of functions that doesn't require authorization: `nuke`
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L11-L11), 


#### Recommendation

Make contract more decentralized.

### `public` functions not called by the contract should be declared `external` instead
In Solidity, function visibility is an important aspect that determines how and where a function can be called from. Two commonly used visibilities are `public` and `external`. A `public` function can be called both from other functions inside the same contract and from outside transactions, while an `external` function can only be called from outside the contract.
A potential pitfall in smart contract development is the misuse of the `public` keyword for functions that are only meant to be accessed externally. When a function is not used internally within a contract and is only intended for external calls, it should be labeled as `external` rather than `public`. Using `public` unnecessarily can introduce potential vulnerabilities and also make the contract consume more gas than required. This is because `public` functions have to add additional code to handle both internal and external calls, while `external` functions can be more optimized since they only handle external calls.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

130:  function getGeneration() public view returns (uint256) {	// @audit-issue

181:  function mintToken(	// @audit-issue

202:  function mintWithBudget(	// @audit-issue

254:  function getTokenLastTransferredTimestamp(	// @audit-issue

264:  function getTokenCreationTimestamp(	// @audit-issue

274:  function isForger(uint256 tokenId) public view returns (bool) {	// @audit-issue
```
[130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L130-L130), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L181), [202](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L202), [254](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L254-L254), [264](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L264-L264), [274](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L274-L274), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

67:  function listForForging(	// @audit-issue
```
[67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L67-L67), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

38:  function listNFTForSale(	// @audit-issue

94:  function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue
```
[38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L38-L38), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L94-L94), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

105:  function getFundBalance() public view returns (uint256) {	// @audit-issue

113:  function getAgeMultiplier() public view returns (uint256) {	// @audit-issue

153:  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue
```
[105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L105-L105), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L113-L113), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L153-L153), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

47:  function writeEntropyBatch1() public {	// @audit-issue

64:  function writeEntropyBatch2() public {	// @audit-issue

84:  function writeEntropyBatch3() public {	// @audit-issue

101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {	// @audit-issue

123:  function getPublicEntropy(	// @audit-issue

131:  function getLastInitializedIndex() public view returns (uint256) {	// @audit-issue

136:  function deriveTokenParameters(	// @audit-issue
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L47-L47), [64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L64-L64), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L84-L84), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L101-L101), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L123-L123), [131](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L131-L131), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L136), 


#### Recommendation

To optimize gas usage and improve code clarity, declare functions that are not called internally within the contract and are intended for external access as `external` rather than `public`. This ensures that these functions are only callable externally, reducing unnecessary gas consumption and potential security risks.

### Functions contain the same code
The functions below have the same implementation as is seen in other files. The functions should be refactored into functions of a common base contract.

```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

27:  function setNukeFundAddress(	// @audit-issue: Seen on line 30 of ./contracts/EntityForging/EntityForging.sol

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue: Seen on line 36 of ./contracts/EntityForging/EntityForging.sol
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L27), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L33), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

63:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue: Seen on line 36 of ./contracts/EntityForging/EntityForging.sol
```
[63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L63), 


#### Recommendation

Identify and consolidate duplicated code blocks in Solidity contracts into reusable modifiers or functions. This approach streamlines the contract by eliminating redundancy, thereby improving clarity, maintainability, and potentially reducing gas costs. For common checks, consider using modifiers for concise and consistent enforcement of conditions. For reusable logic, encapsulate it in functions to avoid code duplication and simplify future updates or bug fixes.

### `if`-statement can be converted to a ternary
The code can be made more compact while also increasing readability by converting the following `if`-statements to ternaries (e.g. `foo += (x > y) ? a : b`)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

52:    if (block.timestamp <= whitelistEndTime) {	// @audit-issue
53:      require(
54:        MerkleProof.verify(proof, rootHash, leaf),
55:        'Not whitelisted user'
56:      );
57:    }

281:    if (generationMintCounts[currentGeneration] >= maxTokensPerGen) {	// @audit-issue
282:      _incrementGeneration();
283:    }

296:    if (!airdropContract.airdropStarted()) {	// @audit-issue
297:      airdropContract.addUserAmount(to, entropyValue);
298:    }

331:    if (	// @audit-issue
332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration
333:    ) {
334:      _incrementGeneration();
335:    }

337:    if (!airdropContract.airdropStarted()) {	// @audit-issue
338:      airdropContract.addUserAmount(newOwner, entropy);
339:    }

377:    if (from != to) {	// @audit-issue
378:      lastTokenTransferredTimestamp[firstTokenId] = block.timestamp;
379:    }
```
[52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L52-L57), [281](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L281-L283), [296](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L296-L298), [331](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L331-L335), [337](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L337-L339), [377](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L377-L379), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

170:    if (	// @audit-issue
171:      slotIndex == slotIndexSelectionPoint &&
172:      numberIndex == numberIndexSelectionPoint
173:    ) {
174:      return 999999;
175:    }
```
[170](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L170-L175), 


#### Recommendation

Consider using single line if statements as ternary.

### Consider using `ERC721A` instead of `ERC721`
[ERC721A](https://www.erc721a.org/) is an implementation of IERC721 with significant gas savings for minting multiple NFTs in a single transaction It was proposed by the Azuki team and used for developing their NFT collection. Compared with `ERC721`, `ERC721A` is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee. [Reference: nextrope](https://nextrope.com/erc721-vs-erc721a-2/) 


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

16:  ERC721Enumerable,	// @audit-issue
```
[16](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L16-L16), 


#### Recommendation

Evaluate the use of 'ERC721A' for minting NFTs, especially when multiple tokens are minted in a single transaction. ERC721A offers significant gas savings compared to standard ERC721, making it a more efficient choice in such scenarios.

### Use a `modifier` instead of `require` to check for `msg.sender`
If some functions are only allowed to be called by some specific users, consider using a `modifier` instead of checking with a `require` statement, especially if this check is done in multiple functions.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

143:      isApprovedOrOwner(msg.sender, tokenId),	// @audit-issue

160:      msg.sender == address(entityForgingContract),	// @audit-issue
```
[143](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L143-L143), [160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L160-L160), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

75:      nftContract.ownerOf(tokenId) == msg.sender,	// @audit-issue

112:      nftContract.ownerOf(mergerTokenId) == msg.sender,	// @audit-issue

116:      nftContract.ownerOf(forgerTokenId) != msg.sender,	// @audit-issue

181:      nftContract.ownerOf(tokenId) == msg.sender ||	// @audit-issue

182:        msg.sender == address(nftContract),	// @audit-issue
```
[75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L75-L75), [112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L112-L112), [116](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L116-L116), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L181-L181), [182](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L182-L182), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

44:      nftContract.ownerOf(tokenId) == msg.sender,	// @audit-issue

49:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue

99:      listing.seller == msg.sender,	// @audit-issue
```
[44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L44-L44), [49](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L49-L49), [99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L99-L99), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

155:      nftContract.isApprovedOrOwner(msg.sender, tokenId),	// @audit-issue

160:        nftContract.isApprovedForAll(msg.sender, address(this)),	// @audit-issue
```
[155](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L155-L155), [160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L160-L160), 


#### Recommendation

To improve code readability and maintainability, consider using a `modifier` to check for `msg.sender` in functions that should only be called by specific users. This not only simplifies the code but also makes it more consistent and easier to audit.

### Consider moving `msg.sender` checks to `modifier`s
If some functions are only allowed to be called by some specific users, consider using a modifier instead of checking with a require statement, especially if this check is done in multiple functions.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

142:    require(	// @audit-issue
143:      isApprovedOrOwner(msg.sender, tokenId),
144:      'ERC721: caller is not token owner or approved'
145:    );

159:    require(	// @audit-issue
160:      msg.sender == address(entityForgingContract),
161:      'unauthorized caller'
162:    );
```
[142](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L142-L145), [159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L159-L162), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

74:    require(	// @audit-issue
75:      nftContract.ownerOf(tokenId) == msg.sender,
76:      'Caller must own the token'
77:    );

111:    require(	// @audit-issue
112:      nftContract.ownerOf(mergerTokenId) == msg.sender,
113:      'Caller must own the merger token'
114:    );

115:    require(	// @audit-issue
116:      nftContract.ownerOf(forgerTokenId) != msg.sender,
117:      'Caller should be different from forger token owner'
118:    );

180:    require(	// @audit-issue
181:      nftContract.ownerOf(tokenId) == msg.sender ||
182:        msg.sender == address(nftContract),
183:      'Caller must own the token'
184:    );
```
[74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L74-L77), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L111-L114), [115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L115-L118), [180](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L180-L184), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

43:    require(	// @audit-issue
44:      nftContract.ownerOf(tokenId) == msg.sender,
45:      'Sender must be the NFT owner.'
46:    );

47:    require(	// @audit-issue
48:      nftContract.getApproved(tokenId) == address(this) ||
49:        nftContract.isApprovedForAll(msg.sender, address(this)),
50:      'Contract must be approved to transfer the NFT.'
51:    );

98:    require(	// @audit-issue
99:      listing.seller == msg.sender,
100:      'Only the seller can canel the listing.'
101:    );
```
[43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L43-L46), [47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L47-L51), [98](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L98-L101), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

154:    require(	// @audit-issue
155:      nftContract.isApprovedOrOwner(msg.sender, tokenId),
156:      'ERC721: caller is not token owner or approved'
157:    );

158:    require(	// @audit-issue
159:      nftContract.getApproved(tokenId) == address(this) ||
160:        nftContract.isApprovedForAll(msg.sender, address(this)),
161:      'Contract must be approved to transfer the NFT.'
162:    );
```
[154](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L154-L157), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L158-L162), 


#### Recommendation

Consider refactoring your code by moving `msg.sender` checks to modifiers when certain functions are only allowed to be called by specific users. This approach can enhance code readability, reduce redundancy, and make it easier to maintain access control logic.

### Constants in comparisons should appear on the left side
Doing so will prevent [typo bugs](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

277:    return roleIndicator == 0;	// @audit-issue
```
[277](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L277-L277), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

92:    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy	// @audit-issue

137:    require(mergerEntropy % 3 != 0, 'Not merger');	// @audit-issue

201:    if (lastForgeResetTimestamp[tokenId] == 0) {	// @audit-issue
```
[92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L92-L92), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L137-L137), [201](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L201-L201), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

33:    require(info.weight == 0, 'Already registered');	// @audit-issue
```
[33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L33-L33), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

76:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

158:    isForger = role == 0;	// @audit-issue

190:    while (number != 0) {	// @audit-issue
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L76-L76), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L158-L158), [190](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L190-L190), 


#### Recommendation

To prevent typo bugs and improve code readability, it's advisable to place constants on the left side of comparisons. This coding practice helps catch accidental assignment (=) instead of comparison (==) and enhances code quality.

### Convert duplicated codes to modifier/functions
Duplicated code in Solidity contracts, especially when repeated across multiple functions, can lead to inefficiencies and increased maintenance challenges. It not only bloats the contract but also makes it harder to implement changes consistently across the codebase. By identifying common patterns or checks that are repeated and abstracting them into modifiers or separate functions, the code can be made more concise, readable, and maintainable. This practice not only reduces the overall bytecode size, potentially lowering deployment and execution costs, but also enhances the contract's reliability by ensuring consistency in how these common checks or operations are executed.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

70:    uint256 endIndex = lastInitializedIndex + batchSize1;	// @audit-issue: Exactly same copy pasted functionality between lines: `{'50->60'}`
71:    unchecked {
72:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
73:        uint256 pseudoRandomValue = uint256(
74:          keccak256(abi.encodePacked(block.number, i))
75:        ) % uint256(10) ** 78;
76:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');
77:        entropySlots[i] = pseudoRandomValue;
78:      }
79:    }
80:    lastInitializedIndex = endIndex;

52:      for (uint256 i = lastInitializedIndex; i < endIndex; i++) {	// @audit-issue: Same for statement in line(s): ['72']
```
[70](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L70-L80), [52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L52-L52), 


#### Recommendation

Identify and consolidate duplicated code blocks in Solidity contracts into reusable modifiers or functions. This approach streamlines the contract by eliminating redundancy, thereby improving clarity, maintainability, and potentially reducing gas costs. For common checks, consider using modifiers for concise and consistent enforcement of conditions. For reusable logic, encapsulate it in functions to avoid code duplication and simplify future updates or bug fixes.

### Style guide: Function ordering does not follow the Solidity style guide
According to the Solidity [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions), functions should be laid out in the following order :`constructor()`, `receive()`, `fallback()`, `external`, `public`, `internal`, `private`, but the cases below do not follow this pattern

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

141:  function burn(uint256 tokenId) external whenNotPaused nonReentrant {	// @audit-issue: burn should come before than getGeneration
```
[141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L141-L141), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

102:  function forgeWithListed(	// @audit-issue: forgeWithListed should come before than listForForging
```
[102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L102), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

63:  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {	// @audit-issue
```
[63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L63-L63), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {	// @audit-issue: setAgeMultplier should come before than getFundBalance
```
[109](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L109), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

206:  function initializeAlphaIndices() public whenNotPaused onlyOwner {	// @audit-issue: initializeAlphaIndices should come before than getEntropy
```
[206](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L206-L206), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html).

### Duplicated `require()`/`revert()` checks should be refactored to a modifier or function
In Solidity contracts, it's common to encounter duplicated `require()` or `revert()` statements across multiple functions. These statements are crucial for validating conditions and ensuring contract integrity. However, repeated checks can lead to code redundancy, making the contract larger and more difficult to maintain. This redundancy can be streamlined by encapsulating the repeated logic in a modifier or a dedicated function. By doing so, the code becomes more concise, easier to audit, and more gas-efficient. Furthermore, centralizing the validation logic in a single location makes the codebase more adaptable to changes and reduces the risk of inconsistencies or errors in future updates.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

235:    require(	// @audit-issue: Same require statement in line(s): ['257', '267']
```
[235](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L235-L235), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

21:        require(success, 'Failed to send Ether to owner');	// @audit-issue: Same require statement in line(s): ['25']

32:    require(weight > 0, 'Invalid weight');	// @audit-issue: Same require statement in line(s): ['42']

43:    require(info.weight > 0, 'Not dev address');	// @audit-issue: Same require statement in line(s): ['53']
```
[21](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L21-L21), [32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L32-L32), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L43-L43), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

48:      require(success, 'ETH send failed');	// @audit-issue: Same require statement in line(s): ['52', '55']

137:    require(	// @audit-issue: Same require statement in line(s): ['186']
```
[48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L48-L48), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L137-L137), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue: Same require statement in line(s): ['76']
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), 


#### Recommendation

Consolidate repeated `require()` or `revert()` checks in Solidity by creating a modifier or a separate function. Apply this modifier to all functions requiring the same validation, or call the function at the beginning of each relevant function. This refactoring enhances contract efficiency, maintainability, and consistency, while potentially reducing gas costs associated with deploying and executing redundant code.

### Adding a return statement when the function defines a named return variable, is redundant
Once the return variable has been assigned (or has its default value), there is no need to explicitly return it at the end of the function, since it's returned automatically.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

136:  function deriveTokenParameters(
137:    uint256 slotIndex,
138:    uint256 numberIndex
139:  )
140:    public
141:    view
142:    returns (
143:      uint256 nukeFactor,
144:      uint256 forgePotential,
145:      uint256 performanceFactor,
146:      bool isForger
147:    )
148:  {
149:    uint256 entropy = getEntropy(slotIndex, numberIndex);
150:
151:    // example calcualtions using entropyto derive game-related parameters
152:    nukeFactor = entropy / 4000000;
153:    forgePotential = getFirstDigit(entropy);
154:    performanceFactor = entropy % 10;
155:
156:    // exmaple logic to determine a boolean property based on entropy
157:    uint256 role = entropy % 3;
158:    isForger = role == 0;
159:
160:    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters	// @audit-issue
161:  }
```
[160](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L161), 


#### Recommendation

When a function defines a named return variable and assigns a value to it, there is no need to add an explicit return statement at the end of the function. The named return variable will be automatically returned with its assigned value. Removing the redundant return statement can lead to cleaner and more concise code, improving readability and reducing the risk of introducing unnecessary errors.

### Consider using `delete` rather than assigning zero to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

351:    generationMintCounts[currentGeneration] = 0;	// @audit-issue
```
[351](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L351-L351), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

204:      forgingCounts[tokenId] = 0; // Reset to the forge potential	// @audit-issue
```
[204](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L204-L204), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

57:    info.weight = 0;	// @audit-issue
```
[57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L57-L57), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

106:      currentNumberIndex = 0;	// @audit-issue

108:        currentSlotIndex = 0;	// @audit-issue
```
[106](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L106-L106), [108](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L108-L108), 


#### Recommendation

When you need to clear or reset values in storage variables, consider using the `delete` keyword instead of manually assigning zero or other values. Using `delete` provides a more explicit and efficient way to clear storage variables. For example, instead of `myVariable = 0;`, you can use `delete myVariable;` to clear the value.

### Too long functions should be refactored
Functions with too many lines are difficult to understand. It is recommended to refactor complex functions into multiple shorter and easier to understand functions.


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

102:  function forgeWithListed(	// @audit-issue 73 lines
```
[102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L102), 


#### Recommendation

To address this issue, refactor long and complex functions into multiple shorter and more manageable functions. This will improve code readability and maintainability, making it easier to understand and maintain your smart contract.

### Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. The solidity style guide recommends a maximumum line length of [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length), so the lines below should be split when they reach that length.        self.impact_details = 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

166:    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; // Calculate the potential claim amount based on the finalNukeFactor	// @audit-issue: Length of the line is: 148

167:    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; // Define a maximum allowed claim amount as 50% of the current fund size	// @audit-issue: Length of the line is: 140

192:    // Assuming tokenAgeInSeconds is the age of the token since it's holding the nft, check if it's over minimum days held	// @audit-issue: Length of the line is: 123
```
[166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L166-L166), [167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L167-L167), [192](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L192-L192), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue: Length of the line is: 129
```
[181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), 


#### Recommendation

To adhere to coding standards and enhance code readability, consider splitting long lines of code when they approach the recommended maximum line length. In Solidity, a common guideline is to limit lines to a maximum of 120 characters. Splitting long lines can improve code maintainability and make it easier to understand.

### Enforce Explicit Bit Size for `uint` and `int` Variables
Adopt best practices by explicitly defining `uint256` and `int256`, instead of using `uint` and `int` without specified bit sizes.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue

375:    uint listedId = entityForgingContract.getListedTokenIds(firstTokenId);	// @audit-issue
```
[114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [375](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L375-L375), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

56:    uint tokenId_	// @audit-issue

57:  ) external view override returns (uint) {	// @audit-issue

62:    uint id	// @audit-issue
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L56-L56), [57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L57-L57), [62](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L62-L62), 


#### Recommendation

Always specify the bit size when declaring `uint` and `int` variables in Solidity. Use `uint256` or `int256` instead of the generic `uint` and `int`. This explicit declaration enhances code readability and clarity, particularly in contracts that utilize various integer sizes. Adhering to this best practice helps in maintaining a clear and unambiguous codebase, making the code more accessible and understandable to developers and auditors alike.

### Avoid the use of sensitive terms
Use [alternative variants](https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/), e.g. allowlist/denylist instead of whitelist/blacklist.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

38:  /// @dev whitelist end time	// @audit-issue: Replace `whitelist` with `allowlist`

39:  uint256 public whitelistEndTime;	// @audit-issue: Replace `whitelist` with `allowlist`

51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {	// @audit-issue: Replace `whitelist` with `allowlist`

52:    if (block.timestamp <= whitelistEndTime) {	// @audit-issue: Replace `whitelist` with `allowlist`

55:        'Not whitelisted user'	// @audit-issue: Replace `whitelist` with `allowlist`

62:    whitelistEndTime = block.timestamp + 24 hours;	// @audit-issue: Replace `whitelist` with `allowlist`

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue: Replace `whitelist` with `allowlist`

127:    whitelistEndTime = endTime_;	// @audit-issue: Replace `whitelist` with `allowlist`

188:    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))	// @audit-issue: Replace `whitelist` with `allowlist`

209:    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))	// @audit-issue: Replace `whitelist` with `allowlist`
```
[38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L38-L38), [39](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L39-L39), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L51-L51), [52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L52-L52), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L55-L55), [62](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L62-L62), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), [127](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L127-L127), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L188-L188), [209](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L209-L209), 


#### Recommendation

To promote inclusive language and avoid insensitive terminology, consider using alternative terms such as 'allowlist' instead of 'whitelist' and 'denylist' instead of 'blacklist.' These alternatives help create a more welcoming and inclusive environment in code and documentation.

### Subtraction may underflow if result of multiplication is too large
The following expressions may underflow, as the subtrahend could be greater than the minuend in case the former is multiplied by a large number.

```solidity
Path: ./contracts/DevFund/DevFund.sol

17:      uint256 remaining = msg.value - (amountPerWeight * totalDevWeight);	// @audit-issue
```
[17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L17-L17), 


#### Recommendation

Implement checks to guard against underflow in subtraction operations that follow multiplications in your Solidity contracts. Use SafeMath library or Solidity 0.8.x or higher, which has built-in overflow/underflow checks, to safely perform these arithmetic operations. Before performing the subtraction, ensure that the minuend is greater than or equal to the subtrahend. If using a version of Solidity that does not include automatic checks, consider adding explicit require statements to validate this condition, or use a library like OpenZeppelin's SafeMath to provide these safeguards. Careful handling of arithmetic operations, especially in critical functions involving financial calculations, is crucial for the security and correctness of your contract.

### Bug in Deduplication of Verbatim Blocks
The current Solidity version has the [Bug in Deduplication of Verbatim Blocks](https://soliditylang.org/blog/2023/11/08/verbatim-invalid-deduplication-bug/) issue.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L2-L2), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L2-L2), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L2-L2), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L2-L2), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L2-L2), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

2:pragma solidity ^0.8.20;	// @audit-issue
```
[2](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L2-L2), 


#### Recommendation

It is recommended to follow the fix provided in the [link](https://soliditylang.org/blog/2023/11/08/verbatim-invalid-deduplication-bug/).

### Use `bytes.concat()` on `bytes` instead of `abi.encodePacked()` for clearer semantic meaning
Starting with version 0.8.4, Solidity has the `bytes.concat()` function, which allows one to concatenate a list of bytes/strings, without extra padding. Using this function rather than `abi.encodePacked()` makes the intended operation more clear, leading to less reviewer confusion.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

54:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

74:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

92:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

208:      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L54-L54), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L74-L74), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L92-L92), [208](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L208-L208), 


#### Recommendation

Consider using `bytes.concat()` instead of `abi.encodePacked()` for concatenating `bytes` for clearer semantic meaning, especially in Solidity versions 0.8.4 and later.

### Use scientific notation (e.g. `1e18`) rather than exponentiation (e.g. `10**18`)
While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

22:  uint256 public maxTokensPerGen = 10000;	// @audit-issue: This number `10000` can be written as `1e4`
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L22-L22), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

12:  uint256 public constant MAX_DENOMINATOR = 100000;	// @audit-issue: This number `100000` can be written as `1e5`
```
[12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L12-L12), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue: This number `1000000` can be written as `1e6`
```
[181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), 


#### Recommendation

Consider using scientific notation (e.g., `1e18`) instead of exponentiation (e.g., `10**18`) for better coding practice. While the compiler can optimize exponentiation, using scientific notation makes the code more readable and relies on standard idioms.

### Hardcoded string that is repeatedly used can be replaced with a constant
For better maintainability, please consider creating and using a constant for those strings instead of hardcoding them.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

77:    require(entityForgingAddress_ != address(0), 'Invalid address');	// @audit-issue: This string `Invalid address` is used also on line(s): `[85, 91]`

197:      (bool refundSuccess, ) = msg.sender.call{ value: excessPayment }('');	// @audit-issue: This string `` is used also on line(s): `[222, 361]`

237:      'ERC721: query for nonexistent token'	// @audit-issue: This string `ERC721: query for nonexistent token` is used also on line(s): `[259, 269]`
```
[77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L77-L77), [197](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L197-L197), [237](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L237-L237), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

76:      'Caller must own the token'	// @audit-issue: This string `Caller must own the token` is used also on line(s): `[183]`

154:      ''	// @audit-issue: This string `` is used also on line(s): `[156, 158]`
```
[76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L76-L76), [154](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L154-L154), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

20:        (bool success, ) = payable(owner()).call{ value: remaining }('');	// @audit-issue: This string `` is used also on line(s): `[24, 89]`

21:        require(success, 'Failed to send Ether to owner');	// @audit-issue: This string `Failed to send Ether to owner` is used also on line(s): `[25]`

32:    require(weight > 0, 'Invalid weight');	// @audit-issue: This string `Invalid weight` is used also on line(s): `[42]`

43:    require(info.weight > 0, 'Not dev address');	// @audit-issue: This string `Not dev address` is used also on line(s): `[53]`
```
[20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L20-L20), [21](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L21-L21), [32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L32-L32), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L43-L43), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

78:      ''	// @audit-issue: This string `` is used also on line(s): `[114]`
```
[78](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L78-L78), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

47:      (bool success, ) = devAddress.call{ value: devShare }('');	// @audit-issue: This string `` is used also on line(s): `[51, 54, 177]`

48:      require(success, 'ETH send failed');	// @audit-issue: This string `ETH send failed` is used also on line(s): `[52, 55]`

139:      'ERC721: operator query for nonexistent token'	// @audit-issue: This string `ERC721: operator query for nonexistent token` is used also on line(s): `[188]`
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L47), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L48-L48), [139](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L139-L139), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue: This string `Invalid value, retry.` is used also on line(s): `[76]`
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), 


#### Recommendation

Identify hardcoded strings that are used repeatedly in your Solidity contract and replace them with constants. Define these constants at the beginning of your contract or in a separate dedicated contract or library if they are shared across multiple contracts. This practice centralizes the management of these values, making your code more maintainable and reducing the risk of inconsistencies or errors when changes are required. Refactoring to use constants not only simplifies updates but also enhances the readability and clarity of your code.

### Style Guide: Surround top level declarations in Solidity source with two blank lines.
1- Surround top level declarations in Solidity source with two blank lines.
2- Within a contract surround function declarations with a single blank line.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

63:  }
64:
65:  // Function to set the nuke fund contract address, restricted to the owner	// @audit-issue: There should be single blank line between function declarations.
66:  function setNukeFundContract(

71:  }
72:
73:  // Function to set the entity merging (breeding) contract address, restricted to the owner	// @audit-issue: There should be single blank line between function declarations.
74:  function setEntityForgingContract(

355:  }
356:
357:  // distributes funds to nukeFund contract, where its then distrubted 10% dev 90% nukeFund	// @audit-issue: There should be single blank line between function declarations.
358:  function _distributeFunds(uint256 totalAmount) private {
```
[65](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L63-L66), [73](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L71-L74), [357](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L355-L358), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

27:  }
28:
29:  // allows the owner to set NukeFund address	// @audit-issue: There should be single blank line between function declarations.
30:  function setNukeFundAddress(
```
[29](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L27-L30), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

24:  }
25:
26:  // allows the owner to set NukeFund address	// @audit-issue: There should be single blank line between function declarations.
27:  function setNukeFundAddress(

35:  }
36:
37:  // function to lsit NFT for sale	// @audit-issue: There should be single blank line between function declarations.
38:  function listNFTForSale(

60:  }
61:
62:  // function to buy an NFT listed for sale	// @audit-issue: There should be single blank line between function declarations.
63:  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {

109:  }
110:
111:  // Correct and secure version of transferToNukeFund function	// @audit-issue: There should be single blank line between function declarations.
112:  function transferToNukeFund(uint256 amount) private {
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L24-L27), [37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L35-L38), [62](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L60-L63), [111](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L109-L112), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

24:  uint256 public ageMultiplier;
25:
26:  // Constructor now properly passes the initial owner address to the Ownable constructor	// @audit-issue: There should be single blank line between function declarations.
27:  constructor(

37:  }
38:
39:  // Fallback function to receive ETH and update fund balance	// @audit-issue: There should be single blank line between function declarations.
40:  receive() external payable {

81:  }
82:
83:  // Allow the owner to update the reference to the ERC721 contract	// @audit-issue: There should be single blank line between function declarations.
84:  function setTraitForgeNftContract(address _traitForgeNft) external onlyOwner {

102:  }
103:
104:  // View function to see the current balance of the fund	// @audit-issue: There should be single blank line between function declarations.
105:  function getFundBalance() public view returns (uint256) {

115:  }
116:
117:  // Calculate the age of a token based on its creation timestamp and current time	// @audit-issue: There should be single blank line between function declarations.
118:  function calculateAge(uint256 tokenId) public view returns (uint256) {

133:  }
134:
135:  // Calculate the nuke factor of a token, which affects the claimable amount from the fund	// @audit-issue: There should be single blank line between function declarations.
136:  function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L24-L27), [39](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L37-L40), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L81-L84), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L102-L105), [117](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L115-L118), [135](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L133-L136), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

22:  address private allowedCaller;
23:
24:  // Modifier to restrict certain functions to the allowed caller	// @audit-issue: There should be single blank line between function declarations.
25:  modifier onlyAllowedCaller() {

33:  }
34:
35:  // Function to update the allowed caller, restricted to the owner of the contract	// @audit-issue: There should be single blank line between function declarations.
36:  function setAllowedCaller(address _allowedCaller) external onlyOwner {

39:  }
40:
41:  // function to get the current allowed caller	// @audit-issue: There should be single blank line between function declarations.
42:  function getAllowedCaller() external view returns (address) {

44:  }
45:
46:  // Functions to initalize entropy values inbatches to spread gas cost over multiple transcations	// @audit-issue: There should be single blank line between function declarations.
47:  function writeEntropyBatch1() public {

61:  }
62:
63:  // second batch initialization	// @audit-issue: There should be single blank line between function declarations.
64:  function writeEntropyBatch2() public {

81:  }
82:
83:  // allows setting a specific entropy slot with a value	// @audit-issue: There should be single blank line between function declarations.
84:  function writeEntropyBatch3() public {

98:  }
99:
100:  // function to retrieve the next entropy value, accessible only by the allowed caller	// @audit-issue: There should be single blank line between function declarations.
101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {

120:  }
121:
122:  // public function to expose entropy calculation for a given slot and number index	// @audit-issue: There should be single blank line between function declarations.
123:  function getPublicEntropy(

128:  }
129:
130:  // function to get the last initialized index for debugging or informational puroposed	// @audit-issue: There should be single blank line between function declarations.
131:  function getLastInitializedIndex() public view returns (uint256) {

133:  }
134:
135:  // function to derive various parameters baed on entrtopy values, demonstrating potential cases	// @audit-issue: There should be single blank line between function declarations.
136:  function deriveTokenParameters(

161:  }
162:
163:  // private function to calculate the entropy value based on slot and number index	// @audit-issue: There should be single blank line between function declarations.
164:  function getEntropy(

185:  }
186:
187:  // Utility function te calcualte the number of digits in a number	// @audit-issue: There should be single blank line between function declarations.
188:  function numberOfDigits(uint256 number) private pure returns (uint256) {

195:  }
196:
197:  // utility to get he first digit of a number	// @audit-issue: There should be single blank line between function declarations.
198:  function getFirstDigit(uint256 number) private pure returns (uint256) {

203:  }
204:
205:  //select index points for 999999, triggered each gen-increment	// @audit-issue: There should be single blank line between function declarations.
206:  function initializeAlphaIndices() public whenNotPaused onlyOwner {
```
[24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L22-L25), [35](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L33-L36), [41](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L39-L42), [46](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L44-L47), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L61-L64), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L81-L84), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L98-L101), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L120-L123), [130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L128-L131), [135](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L133-L136), [163](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L161-L164), [187](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L185-L188), [197](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L195-L198), [205](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L203-L206), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/latest/style-guide.html#blank-lines).

### `constants` should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

62:    whitelistEndTime = block.timestamp + 24 hours;	// @audit-issue
```
[62](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L62-L62), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

137:    require(mergerEntropy % 3 != 0, 'Not merger');	// @audit-issue
```
[137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L137-L137), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

76:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

152:    nukeFactor = entropy / 4000000;	// @audit-issue

154:    performanceFactor = entropy % 10;	// @audit-issue

178:    require(position <= 72, 'Position calculation error');	// @audit-issue

191:      number /= 10;	// @audit-issue

199:    while (number >= 10) {	// @audit-issue

200:      number /= 10;	// @audit-issue
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L76-L76), [152](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L152-L152), [154](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L154-L154), [178](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L178-L178), [191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L191-L191), [199](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L199-L199), [200](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L200-L200), 


#### Recommendation

Consider defining constants with meaningful names for magic numbers and hexadecimal literals to improve code readability and maintainability.

### Contract should expose an `interface`
All `external`/`public` functions should extend an `interface`. This is useful to make sure that the whole API is extracted.


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue
15:  ITraitForgeNft,
16:  ERC721Enumerable,
17:  ReentrancyGuard,
18:  Ownable,
19:  Pausable
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L19), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

10:contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {	// @audit-issue
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L10-L10), 


#### Recommendation

Consider defining an `interface` that includes all `external`/`public` functions of the contract. Exposing a well-defined interface helps ensure that the entire API is extracted and provides a clear and standardized way for other contracts or users to interact with your contract.

### Consider using named returns
Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is ideal for named returns.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

130:  function getGeneration() public view returns (uint256) {	// @audit-issue

134:  function isApprovedOrOwner(
135:    address spender,
136:    uint256 tokenId
137:  ) public view returns (bool) {	// @audit-issue

153:  function forge(
154:    address newOwner,
155:    uint256 parent1Id,
156:    uint256 parent2Id,
157:    string memory
158:  ) external whenNotPaused nonReentrant returns (uint256) {	// @audit-issue

227:  function calculateMintPrice() public view returns (uint256) {	// @audit-issue

234:  function getTokenEntropy(uint256 tokenId) public view returns (uint256) {	// @audit-issue

242:  function getTokenGeneration(uint256 tokenId) public view returns (uint256) {	// @audit-issue

254:  function getTokenLastTransferredTimestamp(
255:    uint256 tokenId
256:  ) public view returns (uint256) {	// @audit-issue

264:  function getTokenCreationTimestamp(
265:    uint256 tokenId
266:  ) public view returns (uint256) {	// @audit-issue

274:  function isForger(uint256 tokenId) public view returns (bool) {	// @audit-issue

311:  function _mintNewEntity(
312:    address newOwner,
313:    uint256 entropy,
314:    uint256 gen
315:  ) private returns (uint256) {	// @audit-issue
```
[130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L130-L130), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L134-L137), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L158), [227](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L227-L227), [234](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L234-L234), [242](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L242-L242), [256](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L254-L256), [266](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L264-L266), [274](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L274-L274), [315](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L311-L315), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

55:  function getListedTokenIds(
56:    uint tokenId_
57:  ) external view override returns (uint) {	// @audit-issue

61:  function getListings(
62:    uint id
63:  ) external view override returns (Listing memory) {	// @audit-issue

102:  function forgeWithListed(
103:    uint256 forgerTokenId,
104:    uint256 mergerTokenId
105:  ) external payable whenNotPaused nonReentrant returns (uint256) {	// @audit-issue
```
[57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L57), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L63), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L105), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

77:  function pendingRewards(address user) external view returns (uint256) {	// @audit-issue

83:  function safeRewardTransfer(
84:    address to,
85:    uint256 amount
86:  ) internal returns (uint256) {	// @audit-issue
```
[77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L77-L77), [86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L86), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

105:  function getFundBalance() public view returns (uint256) {	// @audit-issue

113:  function getAgeMultiplier() public view returns (uint256) {	// @audit-issue

118:  function calculateAge(uint256 tokenId) public view returns (uint256) {	// @audit-issue

136:  function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {	// @audit-issue

184:  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {	// @audit-issue
```
[105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L105-L105), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L113-L113), [118](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L118-L118), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L136-L136), [184](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L184-L184), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

42:  function getAllowedCaller() external view returns (address) {	// @audit-issue

101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {	// @audit-issue

123:  function getPublicEntropy(
124:    uint256 slotIndex,
125:    uint256 numberIndex
126:  ) public view returns (uint256) {	// @audit-issue

131:  function getLastInitializedIndex() public view returns (uint256) {	// @audit-issue

164:  function getEntropy(
165:    uint256 slotIndex,
166:    uint256 numberIndex
167:  ) private view returns (uint256) {	// @audit-issue

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L42-L42), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L101-L101), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L123-L126), [131](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L131-L131), [167](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L164-L167), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), 


#### Recommendation

Adopt named returns in your Solidity functions, especially in cases where functions contain a single return statement. This approach enhances code readability and documentation by making the return values clear and explicit. When defining your function, specify the return types with names, and manipulate these named variables directly within your function logic. Additionally, leverage named returns to streamline your NatSpec documentation, providing clear descriptions for each return variable. Evaluate your current contracts for opportunities to refactor functions to use named returns, prioritizing those with simple return patterns for immediate benefits in gas efficiency and code clarity.

### Inefficient Array Usage
Use mappings instead of arrays for managing lists of data in order to conserve gas. Mappings are less expensive and more efficient for accessing any value without having to iterate through an array.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

182:    bytes32[] calldata proof	// @audit-issue

203:    bytes32[] calldata proof	// @audit-issue
```
[182](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L182-L182), [203](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L203-L203), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

10:  uint256[770] private entropySlots; // Array to store entropy values	// @audit-issue
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L10-L10), 


#### Recommendation

In scenarios where data access efficiency is critical, prefer using mappings over arrays in Solidity contracts. Mappings offer more efficient and gas-effective data retrieval and updates, especially when dealing with large or frequently accessed datasets. Ensure to structure your data and choose keys thoughtfully to maximize the efficiency gains offered by mappings. While arrays might be suitable for ordered data or when the entire dataset needs to be iterated, for most other use cases, mappings are likely to be the more gas-efficient choice.

### Lack of specific import identifier
It is better to use `import {<identifier>} from "<file.sol>"` instead of `import "<file.sol>"` to improve readability and speed up the compilation time.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

7:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

8:import '@openzeppelin/contracts/utils/cryptography/MerkleProof.sol';	// @audit-issue

9:import './ITraitForgeNft.sol';	// @audit-issue

10:import '../EntityForging/IEntityForging.sol';	// @audit-issue

11:import '../EntropyGenerator/IEntropyGenerator.sol';	// @audit-issue

12:import '../Airdrop/IAirdrop.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L8-L8), [9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L9-L9), [10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L10-L10), [11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L12-L12), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

4:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

5:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

7:import './IEntityForging.sol';	// @audit-issue

8:import '../TraitForgeNft/ITraitForgeNft.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L8-L8), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

7:import './IDevFund.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L7-L7), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

7:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

8:import './IEntityTrading.sol';	// @audit-issue

9:import '../TraitForgeNft/ITraitForgeNft.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L8-L8), [9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L9-L9), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

7:import './INukeFund.sol';	// @audit-issue

8:import '../TraitForgeNft/ITraitForgeNft.sol';	// @audit-issue

9:import '../Airdrop/IAirdrop.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L8-L8), [9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

6:import './IEntropyGenerator.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L6-L6), 


#### Recommendation

To improve code clarity and avoid naming conflicts, it's recommended to use specific import identifiers when importing from other contracts or libraries. Instead of using `import "<file.sol>";`, specify the desired identifiers using `import { <identifier1>, <identifier2> } from "<file.sol>";`. This not only enhances readability but also can speed up compilation times by only importing the necessary symbols.

### Returning a struct instead of returning many variables is better
If a function returns [too many variables](https://docs.soliditylang.org/en/latest/contracts.html#returning-multiple-values), replacing them with a struct can improve code readability, maintainability and reusability.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

136:  function deriveTokenParameters(
137:    uint256 slotIndex,
138:    uint256 numberIndex
139:  )
140:    public
141:    view
142:    returns (
143:      uint256 nukeFactor,
144:      uint256 forgePotential,
145:      uint256 performanceFactor,
146:      bool isForger
147:    )	// @audit-issue
```
[147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L147), 


#### Recommendation

Consider returning a struct instead of multiple variables when a function returns many variables. This can enhance code readability, maintainability, and reusability, as well as make the contract interface more user-friendly.

### Ensure Events Emission Prior to External Calls to Prevent Out-of-Order Issues
It's essential to ensure that events follow the best practice of check-effects-interaction and are emitted before any external calls to prevent out-of-order events due to reentrancy. Emitting events post external interactions may cause them to be out of order due to reentrancy, which can be misleading or erroneous for event listeners. [Refer to the Solidity Documentation for best practices](https://solidity.readthedocs.io/en/latest/security-considerations.html#reentrancy).

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

361:    (bool success, ) = nukeFundAddress.call{ value: totalAmount }('');
362:    require(success, 'ETH send failed');
363:
364:    emit FundsDistributedToNukeFund(nukeFundAddress, totalAmount);	// @audit-issue
```
[364](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L361-L364), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

156:    (bool success, ) = nukeFundAddress.call{ value: devFee }('');
157:    require(success, 'Failed to send to NukeFund');
158:    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');
159:    require(success_forge, 'Failed to send to Forge Owner');
160:
161:    // Cancel listed forger nft
162:    _cancelListingForForging(forgerTokenId);
163:
164:    uint256 newEntropy = nftContract.getTokenEntropy(newTokenId);
165:
166:    emit EntityForged(	// @audit-issue
167:      newTokenId,
168:      forgerTokenId,
169:      mergerTokenId,
170:      newEntropy,
171:      forgingFee
172:    );
```
[166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L156-L172), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

20:        (bool success, ) = payable(owner()).call{ value: remaining }('');
21:        require(success, 'Failed to send Ether to owner');
22:      }
23:    } else {
24:      (bool success, ) = payable(owner()).call{ value: msg.value }('');
25:      require(success, 'Failed to send Ether to owner');
26:    }
27:    emit FundReceived(msg.sender, msg.value);	// @audit-issue
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L20-L27), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

77:    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(
78:      ''
79:    );
80:    require(success, 'Failed to send to seller');
81:    nftContract.transferFrom(address(this), msg.sender, tokenId); // transfer NFT to the buyer
82:
83:    delete listings[listedTokenIds[tokenId]]; // remove listing
84:
85:    emit NFTSold(	// @audit-issue
86:      tokenId,
87:      listing.seller,
88:      msg.sender,
89:      msg.value,
90:      nukeFundContribution
91:    ); // emit an event for the sale

114:    (bool success, ) = nukeFundAddress.call{ value: amount }('');
115:    require(success, 'Failed to send Ether to NukeFund');
116:    emit NukeFundContribution(address(this), amount);	// @audit-issue
```
[85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L77-L91), [116](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L114-L116), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

47:      (bool success, ) = devAddress.call{ value: devShare }('');
48:      require(success, 'ETH send failed');
49:      emit DevShareDistributed(devShare);	// @audit-issue

47:      (bool success, ) = devAddress.call{ value: devShare }('');
48:      require(success, 'ETH send failed');
49:      emit DevShareDistributed(devShare);
50:    } else if (!airdropContract.daoFundAllowed()) {
51:      (bool success, ) = payable(owner()).call{ value: devShare }('');
52:      require(success, 'ETH send failed');
53:    } else {
54:      (bool success, ) = daoAddress.call{ value: devShare }('');
55:      require(success, 'ETH send failed');
56:      emit DevShareDistributed(devShare);	// @audit-issue

47:      (bool success, ) = devAddress.call{ value: devShare }('');
48:      require(success, 'ETH send failed');
49:      emit DevShareDistributed(devShare);
50:    } else if (!airdropContract.daoFundAllowed()) {
51:      (bool success, ) = payable(owner()).call{ value: devShare }('');
52:      require(success, 'ETH send failed');
53:    } else {
54:      (bool success, ) = daoAddress.call{ value: devShare }('');
55:      require(success, 'ETH send failed');
56:      emit DevShareDistributed(devShare);
57:    }
58:
59:    emit FundReceived(msg.sender, msg.value); // Log the received funds	// @audit-issue

47:      (bool success, ) = devAddress.call{ value: devShare }('');
48:      require(success, 'ETH send failed');
49:      emit DevShareDistributed(devShare);
50:    } else if (!airdropContract.daoFundAllowed()) {
51:      (bool success, ) = payable(owner()).call{ value: devShare }('');
52:      require(success, 'ETH send failed');
53:    } else {
54:      (bool success, ) = daoAddress.call{ value: devShare }('');
55:      require(success, 'ETH send failed');
56:      emit DevShareDistributed(devShare);
57:    }
58:
59:    emit FundReceived(msg.sender, msg.value); // Log the received funds
60:    emit FundBalanceUpdated(fund); // Update the fund balance	// @audit-issue

177:    (bool success, ) = payable(msg.sender).call{ value: claimAmount }('');
178:    require(success, 'Failed to send Ether');
179:
180:    emit Nuked(msg.sender, tokenId, claimAmount); // Emit the event with the actual claim amount	// @audit-issue

177:    (bool success, ) = payable(msg.sender).call{ value: claimAmount }('');
178:    require(success, 'Failed to send Ether');
179:
180:    emit Nuked(msg.sender, tokenId, claimAmount); // Emit the event with the actual claim amount
181:    emit FundBalanceUpdated(fund); // Update the fund balance	// @audit-issue
```
[49](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L49), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L56), [59](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L59), [60](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L60), [180](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L177-L180), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L177-L181), 


#### Recommendation

To prevent out-of-order events and ensure consistency, always emit events before making any external calls or interactions within your smart contracts. This practice adheres to the check-effects-interaction pattern and helps provide a clear and accurate event log for event listeners. Following this approach enhances the reliability and predictability of your smart contract's behavior.

### Unnecessary Use of override Keyword
In Solidity version 0.8.8 and later, the use of the override keyword becomes superfluous when a function is overriding solely from an interface and the function isn't present in multiple base contracts. Previously, the override keyword was required as an explicit indication to the compiler. However, this is no longer the case, and the extraneous use of the keyword can make the code less clean and more verbose.
Solidity documentation on [Function Overriding](https://docs.soliditylang.org/en/v0.8.20/contracts.html#function-overriding).


```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

367:  function _beforeTokenTransfer(
368:    address from,
369:    address to,
370:    uint256 firstTokenId,
371:    uint256 batchSize
372:  ) internal virtual override {	// @audit-issue
```
[372](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L367-L372), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

55:  function getListedTokenIds(
56:    uint tokenId_
57:  ) external view override returns (uint) {	// @audit-issue

61:  function getListings(
62:    uint id
63:  ) external view override returns (Listing memory) {	// @audit-issue
```
[57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L57), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L63), 


#### Recommendation

In Solidity versions 0.8.8 and later, the `override` keyword is no longer required for functions that are solely overriding from an interface and not present in multiple base contracts. Removing the unnecessary `override` keyword can make the code cleaner and less verbose.

### Missing events in sensitive functions
Events should be emitted when sensitive changes are made to the contracts, but some functions lack them.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

74:  function setEntityForgingContract(	// @audit-issue

82:  function setEntropyGenerator(	// @audit-issue

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {	// @audit-issue

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {	// @audit-issue

108:  function setPriceIncrementByGen(	// @audit-issue

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue
```
[74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L74-L74), [82](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L82-L82), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L100), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L104), [108](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L108), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

30:  function setNukeFundAddress(	// @audit-issue

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue

40:  function setOneYearInDays(uint256 value) external onlyOwner {	// @audit-issue

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {	// @audit-issue
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L36), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L44), 


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

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {	// @audit-issue
```
[63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L63), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L67-L67), [71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L71-L71), [75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L75-L75), [79](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L79-L79), [109](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L109), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

47:  function writeEntropyBatch1() public {	// @audit-issue

64:  function writeEntropyBatch2() public {	// @audit-issue

84:  function writeEntropyBatch3() public {	// @audit-issue

206:  function initializeAlphaIndices() public whenNotPaused onlyOwner {	// @audit-issue
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L47-L47), [64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L64-L64), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L84-L84), [206](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L206-L206), 


#### Recommendation

To enhance transparency and auditability, ensure that events are emitted when sensitive changes are made to the contracts. Review and update functions that lack event emissions, especially in cases where sensitive operations or state changes occur, to provide a clear record of such actions.

### Include sender information in events
When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the `msg.sender` the events of these types of action will make events much more useful to end users, especially when `msg.sender` is not `tx.origin`.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

70:    emit NukeFundContractUpdated(_nukeFundAddress); // Consider adding an event for this.	// @audit-issue

341:    emit NewEntityMinted(newOwner, newTokenId, gen, entropy);	// @audit-issue

354:    emit GenerationIncremented(currentGeneration);	// @audit-issue

364:    emit FundsDistributedToNukeFund(nukeFundAddress, totalAmount);	// @audit-issue
```
[70](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L70-L70), [341](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L341-L341), [354](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L354-L354), [364](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L364-L364), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

99:    emit ListedForForging(tokenId, fee);	// @audit-issue

166:    emit EntityForged(	// @audit-issue

196:    emit CancelledListingForForging(tokenId); // Emitting with 0 fee to denote cancellation	// @audit-issue
```
[99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L99-L99), [166](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L166-L166), [196](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L196-L196), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

37:    emit AddDev(user, weight);	// @audit-issue

48:    emit UpdateDev(user, weight);	// @audit-issue

58:    emit RemoveDev(user);	// @audit-issue
```
[37](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L37-L37), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L48-L48), [58](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L58-L58), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

116:    emit NukeFundContribution(address(this), amount);	// @audit-issue
```
[116](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L116-L116), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

49:      emit DevShareDistributed(devShare);	// @audit-issue

56:      emit DevShareDistributed(devShare);	// @audit-issue

60:    emit FundBalanceUpdated(fund); // Update the fund balance	// @audit-issue

86:    emit TraitForgeNftAddressUpdated(_traitForgeNft); // Emit an event when the address is updated.	// @audit-issue

91:    emit AirdropAddressUpdated(_airdrop); // Emit an event when the address is updated.	// @audit-issue

96:    emit DevAddressUpdated(account);	// @audit-issue

101:    emit DaoAddressUpdated(account);	// @audit-issue

181:    emit FundBalanceUpdated(fund); // Update the fund balance	// @audit-issue
```
[49](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L49-L49), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L56-L56), [60](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L60-L60), [86](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L86-L86), [91](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L91-L91), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L96-L96), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L101-L101), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L181-L181), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

38:    emit AllowedCallerUpdated(_allowedCaller); // Emit an event for this update.	// @audit-issue

117:    emit EntropyRetrieved(entropy);	// @audit-issue
```
[38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L38-L38), [117](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L117-L117), 


#### Recommendation

To improve the usability and analysis of your smart contract events, consider including the `msg.sender` address as part of the event data. This enables easier filtering and identification of the sender's actions within your contract, providing valuable insights for users and external tools.

### Avoid external calls in modifiers
It is unusual to have external calls in modifiers, and doing so will make reviewers more likely to miss important external interactions. Consider moving the external call to an internal function, and calling that function from the modifier.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

54:        MerkleProof.verify(proof, rootHash, leaf),	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L54-L54), 


#### Recommendation

Refrain from incorporating external calls directly within modifiers. Instead, encapsulate the external call within an internal function and invoke this function from within the body of the functions that use the modifier. This approach enhances code readability and security, making it easier for reviewers and auditors to track external interactions. Additionally, it centralizes external calls, simplifying the management and review of these potentially risky operations. Always ensure external calls are handled with care, implementing checks, balances, and reentrancy guards as necessary to protect your contract from malicious actors and unintended consequences.

### Use `uint256`/`int256` instead of `uint`/`int`
It is recommended to use `uint256`/`int256` instead of `uint`/`int` in function parameters, since they are used for function signatures.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue

375:    uint listedId = entityForgingContract.getListedTokenIds(firstTokenId);	// @audit-issue
```
[114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [375](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L375-L375), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

56:    uint tokenId_	// @audit-issue

57:  ) external view override returns (uint) {	// @audit-issue

62:    uint id	// @audit-issue
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L56-L56), [57](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L57-L57), [62](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L62-L62), 


#### Recommendation

Prefer using 'uint256'/'int256' over 'uint'/'int' in function parameters in Solidity. This practice ensures clarity in function signatures and avoids potential confusion with default size assumptions.

### Control structures do not follow the Solidity Style Guide
Refer to the [Solidity style guide - Control Structures](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#control-structures).

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

181:  function mintToken(
182:    bytes32[] calldata proof
183:  )	// @audit-issue

202:  function mintWithBudget(
203:    bytes32[] calldata proof
204:  )	// @audit-issue

331:    if (
332:      generationMintCounts[gen] >= maxTokensPerGen && gen == currentGeneration	// @audit-issue

385:      if (
386:        listing.tokenId == firstTokenId &&
387:        listing.account == from &&
388:        listing.isListed	// @audit-issue
```
[183](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L183), [204](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L204), [332](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L331-L332), [388](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L385-L388), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

136:  function deriveTokenParameters(
137:    uint256 slotIndex,
138:    uint256 numberIndex
139:  )
140:    public
141:    view
142:    returns (
143:      uint256 nukeFactor,
144:      uint256 forgePotential,
145:      uint256 performanceFactor,
146:      bool isForger
147:    )	// @audit-issue

170:    if (
171:      slotIndex == slotIndexSelectionPoint &&
172:      numberIndex == numberIndexSelectionPoint	// @audit-issue
```
[147](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L147), [172](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L170-L172), 


#### Recommendation

Adhere to the Solidity style guide regarding control structures by avoiding the definition of multiple functions with identical names in a contract. Unique and descriptive function names improve code clarity and prevent potential confusion or errors. Consult [Solidity style guide - Control Structures](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#control-structures) for best practices.

### Add inline comments for unnamed variables
In Solidity, it's not uncommon to encounter functions with unnamed parameters, especially when certain arguments are not used within the function body. While this is syntactically valid, it can lead to confusion and a lack of clarity about the function's intent and design. For better readability and maintainability, it's beneficial to include inline comments for these unnamed parameters. This practice provides context to other developers or auditors, clarifying the purpose or the reason for the exclusion of these parameters. For instance, transforming `function foo(address x, address)` to `function foo(address x, address /* unused */)`` or `function foo(address x, address /* y */)``. This small change can significantly enhance the understandability of the contract.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

153:  function forge(
154:    address newOwner,
155:    uint256 parent1Id,
156:    uint256 parent2Id,
157:    string memory	// @audit-issue: Need inline comments for unnamed parameter.
158:  ) external whenNotPaused nonReentrant returns (uint256) {
```
[157](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L158), 


#### Recommendation

Incorporate inline comments for unnamed parameters in Solidity function definitions. This enhances readability and provides clarity on the function's design. For example, change `function foo(address x, address)` to `function foo(address x, address /* unused */)` or `function foo(address x, address /* y */)` to clearly indicate the purpose or the intentional omission of these parameters.

### Consider making contracts `Upgradeable`
This allows for bugs to be fixed in production, at the expense of significantly increasing centralization.

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

Assess the need for upgradeability in your Solidity contracts based on the project's requirements and lifecycle. If chosen, implement a well-known proxy pattern ensuring rigorous security and governance mechanisms are in place. Be aware of the increased centralization and plan accordingly to mitigate potential risks, such as through decentralized governance models or multi-sig control for upgrade decisions.

### Events that mark critical parameter changes should contain both the old and the new value
This should especially be done if the new value is not required to be different from the old value

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

70:    emit NukeFundContractUpdated(_nukeFundAddress); // Consider adding an event for this.	// @audit-issue
```
[70](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L70-L70), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

96:    emit DevAddressUpdated(account);	// @audit-issue

101:    emit DaoAddressUpdated(account);	// @audit-issue
```
[96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L96-L96), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L101-L101), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

38:    emit AllowedCallerUpdated(_allowedCaller); // Emit an event for this update.	// @audit-issue
```
[38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L38-L38), 


#### Recommendation

To enhance transparency and auditability, ensure that events are emitted when sensitive changes are made to the contracts. Review and update functions that lack event emissions, especially in cases where sensitive operations or state changes occur, to provide a clear record of such actions.

### Non-`external`/`public` function names should begin with an underscore
According to the Solidity Style Guide, Non-external/public function names should begin with an [underscore](https://docs.soliditylang.org/en/latest/style-guide.html#underscore-prefix-for-non-external-functions-and-variables)


```solidity
Path: ./contracts/DevFund/DevFund.sol

83:  function safeRewardTransfer(	// @audit-issue
84:    address to,
85:    uint256 amount
86:  ) internal returns (uint256) {
```
[83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L86), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

112:  function transferToNukeFund(uint256 amount) private {	// @audit-issue
```
[112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L112-L112), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

164:  function getEntropy(	// @audit-issue
165:    uint256 slotIndex,
166:    uint256 numberIndex
167:  ) private view returns (uint256) {

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue
```
[164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L164-L167), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), 


#### Recommendation

To adhere to the Solidity Style Guide, consider prefixing the names of non-`external`/`public` functions with an underscore (_). This naming convention enhances code readability and helps distinguish the visibility of functions.

### Non-`external`/`public` variable names should begin with an underscore
According to the Solidity Style Guide, non-external/public variable names should begin with an underscore


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

23:  mapping(uint256 => uint256) private lastForgeResetTimestamp;	// @audit-issue
```
[23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L23-L23), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

14:  uint256 private fund;	// @audit-issue
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L14-L14), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

10:  uint256[770] private entropySlots; // Array to store entropy values	// @audit-issue

11:  uint256 private lastInitializedIndex = 0; // Indexes to keep track of the initialization and usage of entropy values	// @audit-issue

12:  uint256 private currentSlotIndex = 0;	// @audit-issue

13:  uint256 private currentNumberIndex = 0;	// @audit-issue

14:  uint256 private batchSize1 = 256;	// @audit-issue

15:  uint256 private batchSize2 = 512;	// @audit-issue

17:  uint256 private maxSlotIndex = 770;	// @audit-issue

18:  uint256 private maxNumberIndex = 13;	// @audit-issue

22:  address private allowedCaller;	// @audit-issue
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L10-L10), [11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L12-L12), [13](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L13-L13), [14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L14-L14), [15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L15-L15), [17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L17-L17), [18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L18-L18), [22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L22-L22), 


#### Recommendation

To adhere to the Solidity Style Guide, consider prefixing the names of non-`external`/`public` variables with an underscore (_). This naming convention enhances code readability and helps distinguish the visibility of  variables.

### Large numeric literals should use underscores for readability
In Solidity, as in many programming languages, large numeric literals can be difficult to read and interpret at a glance, especially when they consist of many digits. To improve readability and reduce the likelihood of errors, it's beneficial to use underscores (`_`) as separators in these literals. This practice breaks down long numbers into smaller, more readable segments, similar to how commas are used in conventional numeric notation. For instance, `1000000` can be rewritten as `1_000_000`, making it immediately clear that it represents one million. This method of formatting large numbers enhances code clarity without affecting the actual value of the literals.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

22:  uint256 public maxTokensPerGen = 10000;	// @audit-issue
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L22-L22), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

12:  uint256 public constant MAX_DENOMINATOR = 100000;	// @audit-issue
```
[12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L12-L12), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

76:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

152:    nukeFactor = entropy / 4000000;	// @audit-issue

174:      return 999999;	// @audit-issue

181:    uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits	// @audit-issue
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L76-L76), [152](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L152-L152), [174](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L174-L174), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L181-L181), 


#### Recommendation

For improved readability, consider using underscores in large numeric literals. This can make the numbers easier to parse and understand. For example, instead of writing `1000000000000000000`, you can write `1_000_000_000_000_000_000`.

### Names of `private`/`internal` state variables should be prefixed with an underscore
It is recommended by the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#underscore-prefix-for-non-external-functions-and-variables).

```solidity
Path: ./contracts/EntityForging/EntityForging.sol

23:  mapping(uint256 => uint256) private lastForgeResetTimestamp;	// @audit-issue name should be: _astForgeResetTimestamp
```
[23](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L23-L23), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

14:  uint256 private fund;	// @audit-issue name should be: _und
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L14-L14), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

10:  uint256[770] private entropySlots; // Array to store entropy values	// @audit-issue name should be: _ntropySlots

11:  uint256 private lastInitializedIndex = 0; // Indexes to keep track of the initialization and usage of entropy values	// @audit-issue name should be: _astInitializedIndex

12:  uint256 private currentSlotIndex = 0;	// @audit-issue name should be: _urrentSlotIndex

13:  uint256 private currentNumberIndex = 0;	// @audit-issue name should be: _urrentNumberIndex

14:  uint256 private batchSize1 = 256;	// @audit-issue name should be: _atchSize1

15:  uint256 private batchSize2 = 512;	// @audit-issue name should be: _atchSize2

17:  uint256 private maxSlotIndex = 770;	// @audit-issue name should be: _axSlotIndex

18:  uint256 private maxNumberIndex = 13;	// @audit-issue name should be: _axNumberIndex

22:  address private allowedCaller;	// @audit-issue name should be: _llowedCaller
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L10-L10), [11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L12-L12), [13](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L13-L13), [14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L14-L14), [15](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L15-L15), [17](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L17-L17), [18](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L18-L18), [22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L22-L22), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### Using underscore at the end of variable name
The use of the underscore at the end of the variable name is unusual, consider refactoring it.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

75:    address entityForgingAddress_	// @audit-issue

83:    address entropyGeneratorAddress_	// @audit-issue

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue
```
[75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L75-L75), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L83-L83), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

56:    uint tokenId_	// @audit-issue
```
[56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L56-L56), 


#### Recommendation

Refrain from using an underscore at the end of variable names in Solidity. Opt for more conventional naming conventions to enhance code readability and maintainability.

### Employ Explicit Casting to Bytes or Bytes32 for Enhanced Code Clarity and Meaning
Smart contracts are complex entities, and clarity in their operations is fundamental to ensure that they function as intended. Casting a single argument instead of utilizing 'abi.encodePacked()' improves the transparency of the operation. It elucidates the intent of the code, reducing ambiguity and making it easier for auditors and developers to understand the code’s purpose. Such practices promote readability and maintainability, thus reducing the likelihood of errors and misunderstandings. Therefore, it's recommended to employ explicit casts for single arguments where possible, to increase the contract's comprehensibility and ensure a smoother review process.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

54:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

74:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

92:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

208:      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L54-L54), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L74-L74), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L92-L92), [208](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L208-L208), 


#### Recommendation

Consider using `bytes.concat()` instead of `abi.encodePacked()` for concatenating `bytes` for clearer semantic meaning, especially in Solidity versions 0.8.4 and later.

### Revert statements within external and public functions can be used to perform DOS attacks
In Solidity, `revert` statements are used to undo changes and throw an exception when certain conditions are not met. However, in public and external functions, improper use of `revert` can be exploited for Denial of Service (DoS) attacks. An attacker can intentionally trigger these `revert' conditions, causing legitimate transactions to consistently fail. For example, if a function relies on specific conditions from user input or contract state, an attacker could manipulate these to continually force `revert`s, blocking the function's execution. Therefore, it's crucial to design contract logic to handle exceptions properly and avoid scenarios where `revert` can be predictably triggered by malicious actors. This includes careful input validation and considering alternative design patterns that are less susceptible to such abuses.

```solidity
Path: ./contracts/DevFund/DevFund.sol

21:        require(success, 'Failed to send Ether to owner');	// @audit-issue

25:      require(success, 'Failed to send Ether to owner');	// @audit-issue
```
[21](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L21-L21), [25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L25-L25), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

48:      require(success, 'ETH send failed');	// @audit-issue

52:      require(success, 'ETH send failed');	// @audit-issue

55:      require(success, 'ETH send failed');	// @audit-issue
```
[48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L48-L48), [52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L52-L52), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L55-L55), 


#### Recommendation

Design your Solidity contract's public and external functions with care to mitigate the risk of DoS attacks via `revert` statements. Implement robust input validation to ensure inputs are within expected bounds and conditions. Consider alternative logic or design patterns that reduce the reliance on `revert` for critical operations, particularly those that can be influenced externally. Evaluate the use of modifiers, try-catch blocks, or state checks that allow for safer handling of conditions and exceptions. Ensure that your contract's critical functionality remains accessible and resilient against potential abuse of `revert` behavior by malicious actors.

### Use `string.concat()` on strings instead of `abi.encodePacked()` for clearer semantic meaning
From Solidity 0.8.12 onwards, developers can utilize `string.concat()` to concatenate strings without additional padding. Opting for `string.concat()` over `abi.encodePacked()` offers clearer semantic interpretation of the code's intent, enhancing readability. This shift minimizes ambiguity, reducing the potential for misinterpretation by reviewers or future developers. Thus, for string concatenation tasks, it's recommended to transition to `string.concat()` for transparent, straightforward code that communicates its purpose distinctly.

```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

54:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

74:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

92:          keccak256(abi.encodePacked(block.number, i))	// @audit-issue

208:      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))	// @audit-issue
```
[54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L54-L54), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L74-L74), [92](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L92-L92), [208](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L208-L208), 


#### Recommendation

Consider using `bytes.concat()` instead of `abi.encodePacked()` for concatenating `bytes` for clearer semantic meaning, especially in Solidity versions 0.8.4 and later.

### Consider only defining one library/interface/contract per sol file
Combining multiple libraries, interfaces, or contracts in a single .sol file can lead to clutter, reduced readability, and versioning issues. Resolution: Adopt the best practice of defining only one library, interface, or contract per Solidity file. This modular approach enhances clarity, simplifies unit testing, and streamlines code review. Furthermore, segregating components makes version management easier, as updates to one component won't necessitate changes to a file housing multiple unrelated components. Structured file management can further assist in avoiding naming collisions and ensure smoother integration into larger systems or DApps.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol';	// @audit-issue
5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';
6:import '@openzeppelin/contracts/access/Ownable.sol';
7:import '@openzeppelin/contracts/security/Pausable.sol';
8:import '@openzeppelin/contracts/utils/cryptography/MerkleProof.sol';
9:import './ITraitForgeNft.sol';
10:import '../EntityForging/IEntityForging.sol';
11:import '../EntropyGenerator/IEntropyGenerator.sol';
12:import '../Airdrop/IAirdrop.sol';
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L4-L12), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

4:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue
5:import '@openzeppelin/contracts/access/Ownable.sol';
6:import '@openzeppelin/contracts/security/Pausable.sol';
7:import './IEntityForging.sol';
8:import '../TraitForgeNft/ITraitForgeNft.sol';
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L4-L8), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue
5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';
6:import '@openzeppelin/contracts/security/Pausable.sol';
7:import './IDevFund.sol';
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L4-L7), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol';	// @audit-issue
5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';
6:import '@openzeppelin/contracts/access/Ownable.sol';
7:import '@openzeppelin/contracts/security/Pausable.sol';
8:import './IEntityTrading.sol';
9:import '../TraitForgeNft/ITraitForgeNft.sol';
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L4-L9), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue
5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';
6:import '@openzeppelin/contracts/security/Pausable.sol';
7:import './INukeFund.sol';
8:import '../TraitForgeNft/ITraitForgeNft.sol';
9:import '../Airdrop/IAirdrop.sol';
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L4-L9), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue
5:import '@openzeppelin/contracts/security/Pausable.sol';
6:import './IEntropyGenerator.sol';
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L4-L6), 


#### Recommendation

Adopt a modular file structure in your Solidity projects by defining only one library, interface, or contract per file. This approach significantly enhances the clarity and readability of your code, making it easier to manage, test, and review. It also simplifies version control, as updates to individual components are isolated to their respective files, reducing the risk of unintended side effects. Organize your project directory to reflect this structure, with a clear naming convention that matches file names to their contained contracts, libraries, or interfaces. This organization not only prevents naming collisions but also facilitates smoother integration into larger systems or decentralized applications (DApps).

### Reduce deployment costs by tweaking contracts' metadata
When solidity generates the bytecode for the smart contract to be deployed, it appends metadata about the compilation at the end of the bytecode.
By default, the solidity compiler appends metadata at the end of the “actual” initcode, which gets stored to the blockchain when the constructor finishes executing.
Consider tweaking the metadata to avoid this unnecessary allocation. A full guide can be found [here](https://www.rareskills.io/post/solidity-metadata).
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

See [this](https://www.rareskills.io/post/solidity-metadata) link, at its bottom, for full details

### All verbatim blocks are considered identical by deduplicator and can incorrectly be unified
The Solidity Team reported a bug on October 24, 2023, affecting Yul code using the verbatim builtin, specifically in the Block Deduplicator optimizer step. This bug, present since Solidity version 0.8.5, caused incorrect deduplication of verbatim assembly items surrounded by identical opcodes, considering them identical regardless of their data. The bug was confined to pure Yul compilation with optimization enabled and was unlikely to be exploited as an attack vector. The conditions triggering the bug were very specific, and its occurrence was deemed to have a low likelihood. The bug was rated with an overall low score due to these factors.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

Review and assess any Solidity contracts, especially those involving Yul code, that may be impacted by this deduplication bug. If your contracts rely on the Block Deduplicator optimizer and use verbatim blocks in a way that could be affected by this issue, consider updating your Solidity version to one where this bug is fixed, or adjust your contract to avoid this specific scenario. Stay informed about updates from the Solidity Team regarding this and similar issues, and regularly update your Solidity compiler to the latest version to benefit from bug fixes and optimizations. Given the specific and limited nature of this bug, its impact may be minimal, but caution is advised for contracts with complex assembly code or those heavily reliant on optimizer behaviors.

### Consider adding formal verification proofs
Formal verification is the act of proving or disproving the correctness of intended algorithms underlying a system with respect to a certain formal specification/property/invariant, using formal methods of mathematics.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

Consider integrating formal verification into your Solidity contract development process. This can be done by defining formal specifications and properties that your contract should adhere to and using mathematical methods to verify these aspects. Tools and platforms like Certora Prover, Scribble, or OpenZeppelin's test environment can assist in this process. Formal verification should complement traditional testing and auditing methods, offering an additional layer of security assurance. Keep in mind that formal verification requires a thorough understanding of mathematical logic and contract specifications, so it may necessitate additional resources or expertise. Nevertheless, the investment in formal verification can significantly enhance the trustworthiness and robustness of your smart contracts.

### Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

Consider writing test cases.

### Large or complicated code bases should implement invariant tests
This includes: large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers may help significantly.
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

Consider writing invariant test cases.

### Consider adding formal verification proofs
Consider using formal verification to mathematically prove that your code does what is intended, and does not have any edge cases with unexpected behavior. The solidity compiler itself has this functionality [built in](https://docs.soliditylang.org/en/latest/smtchecker.html#smtchecker-and-formal-verification)
```solidity
/// @audit Global finding.
```
[1](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L1), 


#### Recommendation

Consider using formal verification.

### Imports should use double quotes rather than single quotes
In Solidity, the import statement is used to include external files and contracts into the current contract file. While Solidity supports both single (`'`) and double (`"`) quotes for specifying paths in import statements, adopting a consistent style improves code readability and maintainability. The preference for double quotes in import statements aligns with conventions in many programming languages and development environments. Using double quotes consistently for imports and other string literals can help standardize code formatting across Solidity projects, making it easier for developers to read and understand code at a glance.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

7:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

8:import '@openzeppelin/contracts/utils/cryptography/MerkleProof.sol';	// @audit-issue

9:import './ITraitForgeNft.sol';	// @audit-issue

10:import '../EntityForging/IEntityForging.sol';	// @audit-issue

11:import '../EntropyGenerator/IEntropyGenerator.sol';	// @audit-issue

12:import '../Airdrop/IAirdrop.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L8-L8), [9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L9-L9), [10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L10-L10), [11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L11-L11), [12](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L12-L12), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

4:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

5:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

7:import './IEntityForging.sol';	// @audit-issue

8:import '../TraitForgeNft/ITraitForgeNft.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L8-L8), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

7:import './IDevFund.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L7-L7), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

4:import '@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

7:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

8:import './IEntityTrading.sol';	// @audit-issue

9:import '../TraitForgeNft/ITraitForgeNft.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L8-L8), [9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L9-L9), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/ReentrancyGuard.sol';	// @audit-issue

6:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

7:import './INukeFund.sol';	// @audit-issue

8:import '../TraitForgeNft/ITraitForgeNft.sol';	// @audit-issue

9:import '../Airdrop/IAirdrop.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L6-L6), [7](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L7-L7), [8](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L8-L8), [9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

4:import '@openzeppelin/contracts/access/Ownable.sol';	// @audit-issue

5:import '@openzeppelin/contracts/security/Pausable.sol';	// @audit-issue

6:import './IEntropyGenerator.sol';	// @audit-issue
```
[4](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L4-L4), [5](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L5-L5), [6](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L6-L6), 


#### Recommendation

Adopt the practice of using double quotes for all import statements in your Solidity contracts. Review your existing contracts and refactor import statements to use double quotes, `"`, instead of single quotes, `'`. This change contributes to a uniform coding style, enhancing the overall readability and professionalism of your codebase. Consider implementing a linter or formatter within your development workflow to automatically enforce this and other coding standards, ensuring consistency across your project's Solidity files.

### Strings should use double quotes rather than single quotes
In Solidity, when working with strings, it is recommended to use double quotes rather than single quotes. Although both types of quotes can be used interchangeably in Solidity, using double quotes for strings has several advantages.

Firstly, using double quotes for strings helps maintain consistency and readability in your codebase. It provides a clear distinction between string literals and single characters. Additionally, it aligns with the general convention in most programming languages, making your code more familiar to other developers.
See the Solidity Style [Guide](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#other-recommendations).

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

55:        'Not whitelisted user'	// @audit-issue

77:    require(entityForgingAddress_ != address(0), 'Invalid address');	// @audit-issue

85:    require(entropyGeneratorAddress_ != address(0), 'Invalid address');	// @audit-issue

91:    require(airdrop_ != address(0), 'Invalid address');	// @audit-issue

144:      'ERC721: caller is not token owner or approved'	// @audit-issue

161:      'unauthorized caller'	// @audit-issue

191:    require(msg.value >= mintPrice, 'Insufficient ETH send for minting.');	// @audit-issue

197:      (bool refundSuccess, ) = msg.sender.call{ value: excessPayment }('');	// @audit-issue

198:      require(refundSuccess, 'Refund of excess payment failed.');	// @audit-issue

222:      (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');	// @audit-issue

223:      require(refundSuccess, 'Refund failed.');	// @audit-issue

237:      'ERC721: query for nonexistent token'	// @audit-issue

259:      'ERC721: query for nonexistent token'	// @audit-issue

269:      'ERC721: query for nonexistent token'	// @audit-issue

318:      'Exceeds maxTokensPerGen'	// @audit-issue

348:      'Generation limit not yet reached'	// @audit-issue

359:    require(address(this).balance >= totalAmount, 'Insufficient balance');	// @audit-issue

361:    (bool success, ) = nukeFundAddress.call{ value: totalAmount }('');	// @audit-issue

362:    require(success, 'ETH send failed');	// @audit-issue

394:    require(!paused(), 'ERC721Pausable: token transfer while paused');	// @audit-issue
```
[55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L55-L55), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L77-L77), [85](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L85-L85), [91](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L91-L91), [144](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L144-L144), [161](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L161-L161), [191](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L191-L191), [197](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L197-L197), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L198-L198), [222](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L222-L222), [223](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L223-L223), [237](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L237-L237), [259](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L259-L259), [269](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L269-L269), [318](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L318-L318), [348](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L348-L348), [359](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L359-L359), [361](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L361-L361), [362](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L362-L362), [394](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L394-L394), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

73:    require(!_listingInfo.isListed, 'Token is already listed for forging');	// @audit-issue

76:      'Caller must own the token'	// @audit-issue

80:      'Fee should be higher than minimum listing fee'	// @audit-issue

89:      'Entity has reached its forging limit'	// @audit-issue

93:    require(isForger, 'Only forgers can list for forging');	// @audit-issue

113:      'Caller must own the merger token'	// @audit-issue

117:      'Caller should be different from forger token owner'	// @audit-issue

122:      'Invalid token generation'	// @audit-issue

126:    require(msg.value >= forgingFee, 'Insufficient fee for forging');	// @audit-issue

137:    require(mergerEntropy % 3 != 0, 'Not merger');	// @audit-issue

143:      'forgePotential insufficient'	// @audit-issue

154:      ''	// @audit-issue

156:    (bool success, ) = nukeFundAddress.call{ value: devFee }('');	// @audit-issue

157:    require(success, 'Failed to send to NukeFund');	// @audit-issue

158:    (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');	// @audit-issue

159:    require(success_forge, 'Failed to send to Forge Owner');	// @audit-issue

183:      'Caller must own the token'	// @audit-issue

187:      'Token not listed for forging'	// @audit-issue
```
[73](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L73-L73), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L76-L76), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L80-L80), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L89-L89), [93](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L93-L93), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L113-L113), [117](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L117-L117), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L126-L126), [137](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L137-L137), [143](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L143-L143), [154](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L154-L154), [156](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L156-L156), [157](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L157-L157), [158](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L158-L158), [159](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L159-L159), [183](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L183-L183), [187](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L187-L187), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

20:        (bool success, ) = payable(owner()).call{ value: remaining }('');	// @audit-issue

21:        require(success, 'Failed to send Ether to owner');	// @audit-issue

24:      (bool success, ) = payable(owner()).call{ value: msg.value }('');	// @audit-issue

25:      require(success, 'Failed to send Ether to owner');	// @audit-issue

32:    require(weight > 0, 'Invalid weight');	// @audit-issue

33:    require(info.weight == 0, 'Already registered');	// @audit-issue

42:    require(weight > 0, 'Invalid weight');	// @audit-issue

43:    require(info.weight > 0, 'Not dev address');	// @audit-issue

53:    require(info.weight > 0, 'Not dev address');	// @audit-issue

89:    (bool success, ) = payable(to).call{ value: amount }('');	// @audit-issue

90:    require(success, 'Failed to send Reward');	// @audit-issue
```
[20](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L20-L20), [21](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L21-L21), [24](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L24-L24), [25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L25-L25), [32](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L32-L32), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L33-L33), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L42-L42), [43](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L43-L43), [53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L53-L53), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L89-L89), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L90-L90), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

42:    require(price > 0, 'Price must be greater than zero');	// @audit-issue

45:      'Sender must be the NFT owner.'	// @audit-issue

50:      'Contract must be approved to transfer the NFT.'	// @audit-issue

67:      'ETH sent does not match the listing price'	// @audit-issue

69:    require(listing.seller != address(0), 'NFT is not listed for sale.');	// @audit-issue

78:      ''	// @audit-issue

80:    require(success, 'Failed to send to seller');	// @audit-issue

100:      'Only the seller can canel the listing.'	// @audit-issue

102:    require(listing.isActive, 'Listing is not active.');	// @audit-issue

113:    require(nukeFundAddress != address(0), 'NukeFund address not set');	// @audit-issue

114:    (bool success, ) = nukeFundAddress.call{ value: amount }('');	// @audit-issue

115:    require(success, 'Failed to send Ether to NukeFund');	// @audit-issue
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L42-L42), [45](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L45-L45), [50](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L50-L50), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L67-L67), [69](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L69-L69), [78](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L78-L78), [80](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L80-L80), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L100-L100), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L102-L102), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L113-L113), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L114-L114), [115](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L115-L115), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

47:      (bool success, ) = devAddress.call{ value: devShare }('');	// @audit-issue

48:      require(success, 'ETH send failed');	// @audit-issue

51:      (bool success, ) = payable(owner()).call{ value: devShare }('');	// @audit-issue

52:      require(success, 'ETH send failed');	// @audit-issue

54:      (bool success, ) = daoAddress.call{ value: devShare }('');	// @audit-issue

55:      require(success, 'ETH send failed');	// @audit-issue

119:    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');	// @audit-issue

139:      'ERC721: operator query for nonexistent token'	// @audit-issue

156:      'ERC721: caller is not token owner or approved'	// @audit-issue

161:      'Contract must be approved to transfer the NFT.'	// @audit-issue

163:    require(canTokenBeNuked(tokenId), 'Token is not mature yet');	// @audit-issue

177:    (bool success, ) = payable(msg.sender).call{ value: claimAmount }('');	// @audit-issue

178:    require(success, 'Failed to send Ether');	// @audit-issue

188:      'ERC721: operator query for nonexistent token'	// @audit-issue
```
[47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L47-L47), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L48-L48), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L51-L51), [52](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L52-L52), [54](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L54-L54), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L55-L55), [119](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L119-L119), [139](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L139-L139), [156](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L156-L156), [161](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L161-L161), [163](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L163-L163), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L177-L177), [178](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L178-L178), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L188-L188), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

26:    require(msg.sender == allowedCaller, 'Caller is not allowed');	// @audit-issue

48:    require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');	// @audit-issue

56:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

67:      'Batch 2 not ready or already initialized.'	// @audit-issue

76:        require(pseudoRandomValue != 999999, 'Invalid value, retry.');	// @audit-issue

87:      'Batch 3 not ready or already completed.'	// @audit-issue

102:    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');	// @audit-issue

168:    require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');	// @audit-issue

178:    require(position <= 72, 'Position calculation error');	// @audit-issue
```
[26](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L26-L26), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L48-L48), [56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L56-L56), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L67-L67), [76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L76-L76), [87](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L87-L87), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L102-L102), [168](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L168-L168), [178](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L178-L178), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### Inconsistent spacing in comments
Some lines use `// x` and some use `//x`. The instances below point out the usages that don't follow the majority, within each file

```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

71:    //transfer eth to seller (distribute to nukefund)	// @audit-issue
```
[71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L71-L71), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

205:  //select index points for 999999, triggered each gen-increment	// @audit-issue
```
[205](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L205-L205), 


#### Recommendation

Adopt a consistent style for spacing in comments throughout your Solidity code. Choose either `// comment` or `//comment` and apply it uniformly across the entire codebase. This standardization aids in enhancing the overall readability and professionalism of the code. If working in a team, establish and adhere to a shared style guide to maintain consistency in collaborative projects.

### NatSpec: Body of `if` statement should be placed on a new line
According to the [Solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures), `if` statements whose body contains a single line should look like this: solidity `if (x < 10)     x += 1; `

```solidity
Path: ./contracts/DevFund/DevFund.sol

88:    if (amount > _rewardBalance) amount = _rewardBalance;	// @audit-issue
```
[88](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L88-L88), 


#### Recommendation

Adhere to the Solidity style guide by formatting single-line `if` statements with the body on the same line as the condition. For example, use `if (x < 10) x += 1;` for concise and clear code. This style of formatting enhances readability and maintains consistency with Solidity's recommended best practices. It's particularly effective for simple and short conditional operations within the contract's code.

### NatSpec: Contract declarations should have `@author` tag
In the world of decentralized code, giving credit is key. NatSpec's `@author` tag acknowledges the minds behind the code. It appears this Solidity contract omits the `@author` directive in its NatSpec annotations. Properly attributing code to its contributors not only recognizes effort but also aids in establishing trust and credibility. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue missing `@author` tag
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L14), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

10:contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@author` tag
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L10-L10), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

9:contract DevFund is IDevFund, Ownable, ReentrancyGuard, Pausable {	// @audit-issue missing `@author` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

11:contract EntityTrading is IEntityTrading, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@author` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L11-L11), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

11:contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@author` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L11-L11), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

9:contract EntropyGenerator is IEntropyGenerator, Ownable, Pausable {	// @audit-issue missing `@author` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L9-L9), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@dev` tag
NatSpec comments are a critical part of Solidity's documentation system, designed to help developers and others understand the behavior and purpose of a contract. The `@dev` tag, in particular, provides context and insight into the contract's development considerations. A missing `@dev` comment can lead to misunderstandings about the contract, making it harder for others to contribute to or use the contract effectively. Therefore, it's highly recommended to include `@dev` comments in the documentation to enhance code readability and maintainability. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue missing `@dev` tag
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L14), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

10:contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@dev` tag
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L10-L10), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

9:contract DevFund is IDevFund, Ownable, ReentrancyGuard, Pausable {	// @audit-issue missing `@dev` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

11:contract EntityTrading is IEntityTrading, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@dev` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L11-L11), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

11:contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@dev` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L11-L11), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

9:contract EntropyGenerator is IEntropyGenerator, Ownable, Pausable {	// @audit-issue missing `@dev` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L9-L9), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@notice` tag
The `@notice` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue missing `@notice` tag
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L14), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

10:contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@notice` tag
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L10-L10), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

9:contract DevFund is IDevFund, Ownable, ReentrancyGuard, Pausable {	// @audit-issue missing `@notice` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

11:contract EntityTrading is IEntityTrading, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@notice` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L11-L11), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

11:contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@notice` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L11-L11), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

9:contract EntropyGenerator is IEntropyGenerator, Ownable, Pausable {	// @audit-issue missing `@notice` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L9-L9), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Contract declarations should have `@title` tag
The `@title` is used to explain to users what the contract does. The compiler interprets `///` or `/**` comments [as this tag](https://docs.soliditylang.org/en/latest/natspec-format.html#tags) if one wasn't explicitly provided.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

14:contract TraitForgeNft is	// @audit-issue missing `@title` tag
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L14-L14), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

10:contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@title` tag
```
[10](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L10-L10), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

9:contract DevFund is IDevFund, Ownable, ReentrancyGuard, Pausable {	// @audit-issue missing `@title` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L9-L9), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

11:contract EntityTrading is IEntityTrading, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@title` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L11-L11), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

11:contract NukeFund is INukeFund, ReentrancyGuard, Ownable, Pausable {	// @audit-issue missing `@title` tag
```
[11](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L11-L11), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

9:contract EntropyGenerator is IEntropyGenerator, Ownable, Pausable {	// @audit-issue missing `@title` tag
```
[9](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L9-L9), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Use `@inheritdoc` for overriden functions.
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

367:  function _beforeTokenTransfer(	// @audit-issue missing `@inheritdoc` tag
```
[367](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L367-L367), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

55:  function getListedTokenIds(	// @audit-issue missing `@inheritdoc` tag

61:  function getListings(	// @audit-issue missing `@inheritdoc` tag
```
[55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L55), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L61), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function `@return` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including `@return` tag will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

130:  function getGeneration() public view returns (uint256) {	// @audit-issue missing `@return` tag

134:  function isApprovedOrOwner(	// @audit-issue missing `@return` tag

153:  function forge(	// @audit-issue missing `@return` tag

227:  function calculateMintPrice() public view returns (uint256) {	// @audit-issue missing `@return` tag

234:  function getTokenEntropy(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@return` tag

242:  function getTokenGeneration(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@return` tag

246:  function getEntropiesForTokens(	// @audit-issue missing `@return` tag

254:  function getTokenLastTransferredTimestamp(	// @audit-issue missing `@return` tag

264:  function getTokenCreationTimestamp(	// @audit-issue missing `@return` tag

274:  function isForger(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@return` tag

311:  function _mintNewEntity(	// @audit-issue missing `@return` tag
```
[130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L130-L130), [134](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L134-L134), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L153), [227](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L227-L227), [234](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L234-L234), [242](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L242-L242), [246](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L246-L246), [254](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L254-L254), [264](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L264-L264), [274](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L274-L274), [311](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L311-L311), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

48:  function fetchListings() external view returns (Listing[] memory _listings) {	// @audit-issue missing `@return` tag

55:  function getListedTokenIds(	// @audit-issue missing `@return` tag

61:  function getListings(	// @audit-issue missing `@return` tag

102:  function forgeWithListed(	// @audit-issue missing `@return` tag
```
[48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L48-L48), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L55), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L61), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L102), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

77:  function pendingRewards(address user) external view returns (uint256) {	// @audit-issue missing `@return` tag

83:  function safeRewardTransfer(	// @audit-issue missing `@return` tag
```
[77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L77-L77), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L83), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

105:  function getFundBalance() public view returns (uint256) {	// @audit-issue missing `@return` tag

113:  function getAgeMultiplier() public view returns (uint256) {	// @audit-issue missing `@return` tag

118:  function calculateAge(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@return` tag

136:  function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@return` tag

184:  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@return` tag
```
[105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L105-L105), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L113-L113), [118](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L118-L118), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L136-L136), [184](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L184-L184), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

42:  function getAllowedCaller() external view returns (address) {	// @audit-issue missing `@return` tag

101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {	// @audit-issue missing `@return` tag

123:  function getPublicEntropy(	// @audit-issue missing `@return` tag

131:  function getLastInitializedIndex() public view returns (uint256) {	// @audit-issue missing `@return` tag

136:  function deriveTokenParameters(	// @audit-issue missing `@return` tag

164:  function getEntropy(	// @audit-issue missing `@return` tag

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@return` tag

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@return` tag
```
[42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L42-L42), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L101-L101), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L123-L123), [131](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L131-L131), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L136), [164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L164-L164), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@notice` tag
The `@notice` tag in NatSpec comments is used to provide important explanations to end users about what a function does. It appears that this contract's function declarations are missing `@notice` tags in their NatSpec annotations.

The absence of `@notice` tags reduces the contract's transparency and could lead to misunderstandings about a function's purpose and behavior.  [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

61:  constructor() ERC721('TraitForgeNft', 'TFGNFT') {	// @audit-issue missing `@notice` tag

66:  function setNukeFundContract(	// @audit-issue missing `@notice` tag

74:  function setEntityForgingContract(	// @audit-issue missing `@notice` tag

82:  function setEntropyGenerator(	// @audit-issue missing `@notice` tag

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue missing `@notice` tag

96:  function startAirdrop(uint256 amount) external whenNotPaused onlyOwner {	// @audit-issue missing `@notice` tag

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {	// @audit-issue missing `@notice` tag

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {	// @audit-issue missing `@notice` tag

108:  function setPriceIncrementByGen(	// @audit-issue missing `@notice` tag

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue missing `@notice` tag

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue missing `@notice` tag

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue missing `@notice` tag

130:  function getGeneration() public view returns (uint256) {	// @audit-issue missing `@notice` tag

134:  function isApprovedOrOwner(	// @audit-issue missing `@notice` tag

141:  function burn(uint256 tokenId) external whenNotPaused nonReentrant {	// @audit-issue missing `@notice` tag

153:  function forge(	// @audit-issue missing `@notice` tag

181:  function mintToken(	// @audit-issue missing `@notice` tag

202:  function mintWithBudget(	// @audit-issue missing `@notice` tag

227:  function calculateMintPrice() public view returns (uint256) {	// @audit-issue missing `@notice` tag

234:  function getTokenEntropy(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@notice` tag

242:  function getTokenGeneration(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@notice` tag

246:  function getEntropiesForTokens(	// @audit-issue missing `@notice` tag

254:  function getTokenLastTransferredTimestamp(	// @audit-issue missing `@notice` tag

264:  function getTokenCreationTimestamp(	// @audit-issue missing `@notice` tag

274:  function isForger(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@notice` tag

280:  function _mintInternal(address to, uint256 mintPrice) internal {	// @audit-issue missing `@notice` tag

311:  function _mintNewEntity(	// @audit-issue missing `@notice` tag

345:  function _incrementGeneration() private {	// @audit-issue missing `@notice` tag

358:  function _distributeFunds(uint256 totalAmount) private {	// @audit-issue missing `@notice` tag

367:  function _beforeTokenTransfer(	// @audit-issue missing `@notice` tag
```
[61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L61-L61), [66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L66-L66), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L74-L74), [82](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L82-L82), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L96-L96), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L100), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L104), [108](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L108), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), [130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L130-L130), [134](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L134-L134), [141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L141-L141), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L153), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L181), [202](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L202), [227](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L227-L227), [234](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L234-L234), [242](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L242-L242), [246](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L246-L246), [254](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L254-L254), [264](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L264-L264), [274](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L274-L274), [280](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L280-L280), [311](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L311-L311), [345](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L345-L345), [358](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L358-L358), [367](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L367-L367), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

25:  constructor(address _traitForgeNft) {	// @audit-issue missing `@notice` tag

30:  function setNukeFundAddress(	// @audit-issue missing `@notice` tag

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@notice` tag

40:  function setOneYearInDays(uint256 value) external onlyOwner {	// @audit-issue missing `@notice` tag

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {	// @audit-issue missing `@notice` tag

48:  function fetchListings() external view returns (Listing[] memory _listings) {	// @audit-issue missing `@notice` tag

55:  function getListedTokenIds(	// @audit-issue missing `@notice` tag

61:  function getListings(	// @audit-issue missing `@notice` tag

67:  function listForForging(	// @audit-issue missing `@notice` tag

102:  function forgeWithListed(	// @audit-issue missing `@notice` tag

177:  function cancelListingForForging(	// @audit-issue missing `@notice` tag

193:  function _cancelListingForForging(uint256 tokenId) internal {	// @audit-issue missing `@notice` tag

199:  function _resetForgingCountIfNeeded(uint256 tokenId) private {	// @audit-issue missing `@notice` tag
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L25-L25), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L36), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L44), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L48-L48), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L55), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L61), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L67-L67), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L102), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L177-L177), [193](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L193-L193), [199](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L199-L199), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

14:  receive() external payable {	// @audit-issue missing `@notice` tag

30:  function addDev(address user, uint256 weight) external onlyOwner {	// @audit-issue missing `@notice` tag

40:  function updateDev(address user, uint256 weight) external onlyOwner {	// @audit-issue missing `@notice` tag

51:  function removeDev(address user) external onlyOwner {	// @audit-issue missing `@notice` tag

61:  function claim() external whenNotPaused nonReentrant {	// @audit-issue missing `@notice` tag

77:  function pendingRewards(address user) external view returns (uint256) {	// @audit-issue missing `@notice` tag

83:  function safeRewardTransfer(	// @audit-issue missing `@notice` tag
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L14-L14), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L30-L30), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L40-L40), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L51-L51), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L61-L61), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L77-L77), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L83), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

22:  constructor(address _traitForgeNft) {	// @audit-issue missing `@notice` tag

27:  function setNukeFundAddress(	// @audit-issue missing `@notice` tag

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@notice` tag

38:  function listNFTForSale(	// @audit-issue missing `@notice` tag

63:  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {	// @audit-issue missing `@notice` tag

94:  function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue missing `@notice` tag

112:  function transferToNukeFund(uint256 amount) private {	// @audit-issue missing `@notice` tag
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L22-L22), [27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L27), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L33), [38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L38-L38), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L63-L63), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L94-L94), [112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L112-L112), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

27:  constructor(	// @audit-issue missing `@notice` tag

40:  receive() external payable {	// @audit-issue missing `@notice` tag

63:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@notice` tag

67:  function setMinimumDaysHeld(uint256 value) external onlyOwner {	// @audit-issue missing `@notice` tag

71:  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {	// @audit-issue missing `@notice` tag

75:  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {	// @audit-issue missing `@notice` tag

79:  function setNukeFactorMaxParam(uint256 value) external onlyOwner {	// @audit-issue missing `@notice` tag

84:  function setTraitForgeNftContract(address _traitForgeNft) external onlyOwner {	// @audit-issue missing `@notice` tag

89:  function setAirdropContract(address _airdrop) external onlyOwner {	// @audit-issue missing `@notice` tag

94:  function setDevAddress(address payable account) external onlyOwner {	// @audit-issue missing `@notice` tag

99:  function setDaoAddress(address payable account) external onlyOwner {	// @audit-issue missing `@notice` tag

105:  function getFundBalance() public view returns (uint256) {	// @audit-issue missing `@notice` tag

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {	// @audit-issue missing `@notice` tag

113:  function getAgeMultiplier() public view returns (uint256) {	// @audit-issue missing `@notice` tag

118:  function calculateAge(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@notice` tag

136:  function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@notice` tag

153:  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue missing `@notice` tag

184:  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@notice` tag
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L27-L27), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L40-L40), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L63), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L67-L67), [71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L71-L71), [75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L75-L75), [79](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L79-L79), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L84-L84), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L89-L89), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L94-L94), [99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L99-L99), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L105-L105), [109](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L109), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L113-L113), [118](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L118-L118), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L136-L136), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L153-L153), [184](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L184-L184), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

30:  constructor(address _traitForgetNft) {	// @audit-issue missing `@notice` tag

36:  function setAllowedCaller(address _allowedCaller) external onlyOwner {	// @audit-issue missing `@notice` tag

42:  function getAllowedCaller() external view returns (address) {	// @audit-issue missing `@notice` tag

47:  function writeEntropyBatch1() public {	// @audit-issue missing `@notice` tag

64:  function writeEntropyBatch2() public {	// @audit-issue missing `@notice` tag

84:  function writeEntropyBatch3() public {	// @audit-issue missing `@notice` tag

101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {	// @audit-issue missing `@notice` tag

123:  function getPublicEntropy(	// @audit-issue missing `@notice` tag

131:  function getLastInitializedIndex() public view returns (uint256) {	// @audit-issue missing `@notice` tag

136:  function deriveTokenParameters(	// @audit-issue missing `@notice` tag

164:  function getEntropy(	// @audit-issue missing `@notice` tag

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@notice` tag

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@notice` tag

206:  function initializeAlphaIndices() public whenNotPaused onlyOwner {	// @audit-issue missing `@notice` tag
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L36-L36), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L42-L42), [47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L47-L47), [64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L64-L64), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L84-L84), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L101-L101), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L123-L123), [131](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L131-L131), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L136), [164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L164-L164), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), [206](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L206-L206), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function declarations should have `@dev` tag
Some functions have an incomplete NatSpec: add a `@dev` notation to describe the function to improve the code documentation.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

61:  constructor() ERC721('TraitForgeNft', 'TFGNFT') {	// @audit-issue missing `@dev` tag

66:  function setNukeFundContract(	// @audit-issue missing `@dev` tag

74:  function setEntityForgingContract(	// @audit-issue missing `@dev` tag

82:  function setEntropyGenerator(	// @audit-issue missing `@dev` tag

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue missing `@dev` tag

96:  function startAirdrop(uint256 amount) external whenNotPaused onlyOwner {	// @audit-issue missing `@dev` tag

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {	// @audit-issue missing `@dev` tag

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {	// @audit-issue missing `@dev` tag

108:  function setPriceIncrementByGen(	// @audit-issue missing `@dev` tag

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue missing `@dev` tag

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue missing `@dev` tag

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue missing `@dev` tag

130:  function getGeneration() public view returns (uint256) {	// @audit-issue missing `@dev` tag

134:  function isApprovedOrOwner(	// @audit-issue missing `@dev` tag

141:  function burn(uint256 tokenId) external whenNotPaused nonReentrant {	// @audit-issue missing `@dev` tag

153:  function forge(	// @audit-issue missing `@dev` tag

181:  function mintToken(	// @audit-issue missing `@dev` tag

202:  function mintWithBudget(	// @audit-issue missing `@dev` tag

227:  function calculateMintPrice() public view returns (uint256) {	// @audit-issue missing `@dev` tag

234:  function getTokenEntropy(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@dev` tag

242:  function getTokenGeneration(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@dev` tag

246:  function getEntropiesForTokens(	// @audit-issue missing `@dev` tag

254:  function getTokenLastTransferredTimestamp(	// @audit-issue missing `@dev` tag

264:  function getTokenCreationTimestamp(	// @audit-issue missing `@dev` tag

274:  function isForger(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@dev` tag

280:  function _mintInternal(address to, uint256 mintPrice) internal {	// @audit-issue missing `@dev` tag

311:  function _mintNewEntity(	// @audit-issue missing `@dev` tag

345:  function _incrementGeneration() private {	// @audit-issue missing `@dev` tag

358:  function _distributeFunds(uint256 totalAmount) private {	// @audit-issue missing `@dev` tag

367:  function _beforeTokenTransfer(	// @audit-issue missing `@dev` tag
```
[61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L61-L61), [66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L66-L66), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L74-L74), [82](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L82-L82), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L96-L96), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L100), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L104), [108](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L108), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), [130](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L130-L130), [134](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L134-L134), [141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L141-L141), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L153), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L181), [202](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L202), [227](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L227-L227), [234](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L234-L234), [242](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L242-L242), [246](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L246-L246), [254](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L254-L254), [264](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L264-L264), [274](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L274-L274), [280](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L280-L280), [311](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L311-L311), [345](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L345-L345), [358](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L358-L358), [367](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L367-L367), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

25:  constructor(address _traitForgeNft) {	// @audit-issue missing `@dev` tag

30:  function setNukeFundAddress(	// @audit-issue missing `@dev` tag

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@dev` tag

40:  function setOneYearInDays(uint256 value) external onlyOwner {	// @audit-issue missing `@dev` tag

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {	// @audit-issue missing `@dev` tag

48:  function fetchListings() external view returns (Listing[] memory _listings) {	// @audit-issue missing `@dev` tag

55:  function getListedTokenIds(	// @audit-issue missing `@dev` tag

61:  function getListings(	// @audit-issue missing `@dev` tag

67:  function listForForging(	// @audit-issue missing `@dev` tag

102:  function forgeWithListed(	// @audit-issue missing `@dev` tag

177:  function cancelListingForForging(	// @audit-issue missing `@dev` tag

193:  function _cancelListingForForging(uint256 tokenId) internal {	// @audit-issue missing `@dev` tag

199:  function _resetForgingCountIfNeeded(uint256 tokenId) private {	// @audit-issue missing `@dev` tag
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L25-L25), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L36), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L44), [48](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L48-L48), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L55), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L61), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L67-L67), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L102), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L177-L177), [193](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L193-L193), [199](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L199-L199), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

14:  receive() external payable {	// @audit-issue missing `@dev` tag

30:  function addDev(address user, uint256 weight) external onlyOwner {	// @audit-issue missing `@dev` tag

40:  function updateDev(address user, uint256 weight) external onlyOwner {	// @audit-issue missing `@dev` tag

51:  function removeDev(address user) external onlyOwner {	// @audit-issue missing `@dev` tag

61:  function claim() external whenNotPaused nonReentrant {	// @audit-issue missing `@dev` tag

77:  function pendingRewards(address user) external view returns (uint256) {	// @audit-issue missing `@dev` tag

83:  function safeRewardTransfer(	// @audit-issue missing `@dev` tag
```
[14](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L14-L14), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L30-L30), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L40-L40), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L51-L51), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L61-L61), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L77-L77), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L83), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

22:  constructor(address _traitForgeNft) {	// @audit-issue missing `@dev` tag

27:  function setNukeFundAddress(	// @audit-issue missing `@dev` tag

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@dev` tag

38:  function listNFTForSale(	// @audit-issue missing `@dev` tag

63:  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {	// @audit-issue missing `@dev` tag

94:  function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue missing `@dev` tag

112:  function transferToNukeFund(uint256 amount) private {	// @audit-issue missing `@dev` tag
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L22-L22), [27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L27), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L33), [38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L38-L38), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L63-L63), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L94-L94), [112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L112-L112), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

27:  constructor(	// @audit-issue missing `@dev` tag

40:  receive() external payable {	// @audit-issue missing `@dev` tag

63:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@dev` tag

67:  function setMinimumDaysHeld(uint256 value) external onlyOwner {	// @audit-issue missing `@dev` tag

71:  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {	// @audit-issue missing `@dev` tag

75:  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {	// @audit-issue missing `@dev` tag

79:  function setNukeFactorMaxParam(uint256 value) external onlyOwner {	// @audit-issue missing `@dev` tag

84:  function setTraitForgeNftContract(address _traitForgeNft) external onlyOwner {	// @audit-issue missing `@dev` tag

89:  function setAirdropContract(address _airdrop) external onlyOwner {	// @audit-issue missing `@dev` tag

94:  function setDevAddress(address payable account) external onlyOwner {	// @audit-issue missing `@dev` tag

99:  function setDaoAddress(address payable account) external onlyOwner {	// @audit-issue missing `@dev` tag

105:  function getFundBalance() public view returns (uint256) {	// @audit-issue missing `@dev` tag

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {	// @audit-issue missing `@dev` tag

113:  function getAgeMultiplier() public view returns (uint256) {	// @audit-issue missing `@dev` tag

118:  function calculateAge(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@dev` tag

136:  function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@dev` tag

153:  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue missing `@dev` tag

184:  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@dev` tag
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L27-L27), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L40-L40), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L63), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L67-L67), [71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L71-L71), [75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L75-L75), [79](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L79-L79), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L84-L84), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L89-L89), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L94-L94), [99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L99-L99), [105](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L105-L105), [109](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L109), [113](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L113-L113), [118](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L118-L118), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L136-L136), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L153-L153), [184](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L184-L184), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

30:  constructor(address _traitForgetNft) {	// @audit-issue missing `@dev` tag

36:  function setAllowedCaller(address _allowedCaller) external onlyOwner {	// @audit-issue missing `@dev` tag

42:  function getAllowedCaller() external view returns (address) {	// @audit-issue missing `@dev` tag

47:  function writeEntropyBatch1() public {	// @audit-issue missing `@dev` tag

64:  function writeEntropyBatch2() public {	// @audit-issue missing `@dev` tag

84:  function writeEntropyBatch3() public {	// @audit-issue missing `@dev` tag

101:  function getNextEntropy() public onlyAllowedCaller returns (uint256) {	// @audit-issue missing `@dev` tag

123:  function getPublicEntropy(	// @audit-issue missing `@dev` tag

131:  function getLastInitializedIndex() public view returns (uint256) {	// @audit-issue missing `@dev` tag

136:  function deriveTokenParameters(	// @audit-issue missing `@dev` tag

164:  function getEntropy(	// @audit-issue missing `@dev` tag

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@dev` tag

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@dev` tag

206:  function initializeAlphaIndices() public whenNotPaused onlyOwner {	// @audit-issue missing `@dev` tag
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L36-L36), [42](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L42-L42), [47](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L47-L47), [64](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L64-L64), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L84-L84), [101](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L101-L101), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L123-L123), [131](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L131-L131), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L136), [164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L164-L164), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), [206](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L206-L206), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Function/Constructor `@param` tag is missing
Natural Specification (NatSpec) comments are crucial for understanding the role of function arguments in your Solidity code. Including @param tags will not only improve your code's readability but also its maintainability by clearly defining each argument's purpose. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

66:  function setNukeFundContract(	// @audit-issue missing `@param` tag

74:  function setEntityForgingContract(	// @audit-issue missing `@param` tag

82:  function setEntropyGenerator(	// @audit-issue missing `@param` tag

90:  function setAirdropContract(address airdrop_) external onlyOwner {	// @audit-issue missing `@param` tag

96:  function startAirdrop(uint256 amount) external whenNotPaused onlyOwner {	// @audit-issue missing `@param` tag

100:  function setStartPrice(uint256 _startPrice) external onlyOwner {	// @audit-issue missing `@param` tag

104:  function setPriceIncrement(uint256 _priceIncrement) external onlyOwner {	// @audit-issue missing `@param` tag

108:  function setPriceIncrementByGen(	// @audit-issue missing `@param` tag

114:  function setMaxGeneration(uint maxGeneration_) external onlyOwner {	// @audit-issue missing `@param` tag

122:  function setRootHash(bytes32 rootHash_) external onlyOwner {	// @audit-issue missing `@param` tag

126:  function setWhitelistEndTime(uint256 endTime_) external onlyOwner {	// @audit-issue missing `@param` tag

134:  function isApprovedOrOwner(	// @audit-issue missing `@param` tag

141:  function burn(uint256 tokenId) external whenNotPaused nonReentrant {	// @audit-issue missing `@param` tag

153:  function forge(	// @audit-issue missing `@param` tag

181:  function mintToken(	// @audit-issue missing `@param` tag

202:  function mintWithBudget(	// @audit-issue missing `@param` tag

234:  function getTokenEntropy(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@param` tag

242:  function getTokenGeneration(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@param` tag

246:  function getEntropiesForTokens(	// @audit-issue missing `@param` tag

254:  function getTokenLastTransferredTimestamp(	// @audit-issue missing `@param` tag

264:  function getTokenCreationTimestamp(	// @audit-issue missing `@param` tag

274:  function isForger(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@param` tag

280:  function _mintInternal(address to, uint256 mintPrice) internal {	// @audit-issue missing `@param` tag

311:  function _mintNewEntity(	// @audit-issue missing `@param` tag

358:  function _distributeFunds(uint256 totalAmount) private {	// @audit-issue missing `@param` tag

367:  function _beforeTokenTransfer(	// @audit-issue missing `@param` tag
```
[66](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L66-L66), [74](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L74-L74), [82](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L82-L82), [90](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L90-L90), [96](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L96-L96), [100](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L100-L100), [104](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L104-L104), [108](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L108-L108), [114](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L114-L114), [122](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L122-L122), [126](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L126-L126), [134](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L134-L134), [141](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L141-L141), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L153-L153), [181](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L181-L181), [202](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L202-L202), [234](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L234-L234), [242](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L242-L242), [246](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L246-L246), [254](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L254-L254), [264](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L264-L264), [274](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L274-L274), [280](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L280-L280), [311](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L311-L311), [358](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L358-L358), [367](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L367-L367), 


```solidity
Path: ./contracts/EntityForging/EntityForging.sol

25:  constructor(address _traitForgeNft) {	// @audit-issue missing `@param` tag

30:  function setNukeFundAddress(	// @audit-issue missing `@param` tag

36:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@param` tag

40:  function setOneYearInDays(uint256 value) external onlyOwner {	// @audit-issue missing `@param` tag

44:  function setMinimumListingFee(uint256 _fee) external onlyOwner {	// @audit-issue missing `@param` tag

55:  function getListedTokenIds(	// @audit-issue missing `@param` tag

61:  function getListings(	// @audit-issue missing `@param` tag

67:  function listForForging(	// @audit-issue missing `@param` tag

102:  function forgeWithListed(	// @audit-issue missing `@param` tag

177:  function cancelListingForForging(	// @audit-issue missing `@param` tag

193:  function _cancelListingForForging(uint256 tokenId) internal {	// @audit-issue missing `@param` tag

199:  function _resetForgingCountIfNeeded(uint256 tokenId) private {	// @audit-issue missing `@param` tag
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L25-L25), [30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L36-L36), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L40-L40), [44](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L44-L44), [55](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L55-L55), [61](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L61-L61), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L67-L67), [102](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L102-L102), [177](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L177-L177), [193](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L193-L193), [199](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityForging/EntityForging.sol#L199-L199), 


```solidity
Path: ./contracts/DevFund/DevFund.sol

30:  function addDev(address user, uint256 weight) external onlyOwner {	// @audit-issue missing `@param` tag

40:  function updateDev(address user, uint256 weight) external onlyOwner {	// @audit-issue missing `@param` tag

51:  function removeDev(address user) external onlyOwner {	// @audit-issue missing `@param` tag

77:  function pendingRewards(address user) external view returns (uint256) {	// @audit-issue missing `@param` tag

83:  function safeRewardTransfer(	// @audit-issue missing `@param` tag
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L30-L30), [40](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L40-L40), [51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L51-L51), [77](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L77-L77), [83](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/DevFund/DevFund.sol#L83-L83), 


```solidity
Path: ./contracts/EntityTrading/EntityTrading.sol

22:  constructor(address _traitForgeNft) {	// @audit-issue missing `@param` tag

27:  function setNukeFundAddress(	// @audit-issue missing `@param` tag

33:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@param` tag

38:  function listNFTForSale(	// @audit-issue missing `@param` tag

63:  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {	// @audit-issue missing `@param` tag

94:  function cancelListing(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue missing `@param` tag

112:  function transferToNukeFund(uint256 amount) private {	// @audit-issue missing `@param` tag
```
[22](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L22-L22), [27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L27-L27), [33](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L33-L33), [38](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L38-L38), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L63-L63), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L94-L94), [112](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntityTrading/EntityTrading.sol#L112-L112), 


```solidity
Path: ./contracts/NukeFund/NukeFund.sol

27:  constructor(	// @audit-issue missing `@param` tag

63:  function setTaxCut(uint256 _taxCut) external onlyOwner {	// @audit-issue missing `@param` tag

67:  function setMinimumDaysHeld(uint256 value) external onlyOwner {	// @audit-issue missing `@param` tag

71:  function setDefaultNukeFactorIncrease(uint256 value) external onlyOwner {	// @audit-issue missing `@param` tag

75:  function setMaxAllowedClaimDivisor(uint256 value) external onlyOwner {	// @audit-issue missing `@param` tag

79:  function setNukeFactorMaxParam(uint256 value) external onlyOwner {	// @audit-issue missing `@param` tag

84:  function setTraitForgeNftContract(address _traitForgeNft) external onlyOwner {	// @audit-issue missing `@param` tag

89:  function setAirdropContract(address _airdrop) external onlyOwner {	// @audit-issue missing `@param` tag

94:  function setDevAddress(address payable account) external onlyOwner {	// @audit-issue missing `@param` tag

99:  function setDaoAddress(address payable account) external onlyOwner {	// @audit-issue missing `@param` tag

109:  function setAgeMultplier(uint256 _ageMultiplier) external onlyOwner {	// @audit-issue missing `@param` tag

118:  function calculateAge(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@param` tag

136:  function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {	// @audit-issue missing `@param` tag

153:  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {	// @audit-issue missing `@param` tag

184:  function canTokenBeNuked(uint256 tokenId) public view returns (bool) {	// @audit-issue missing `@param` tag
```
[27](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L27-L27), [63](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L63-L63), [67](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L67-L67), [71](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L71-L71), [75](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L75-L75), [79](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L79-L79), [84](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L84-L84), [89](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L89-L89), [94](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L94-L94), [99](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L99-L99), [109](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L109-L109), [118](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L118-L118), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L136-L136), [153](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L153-L153), [184](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/NukeFund/NukeFund.sol#L184-L184), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

30:  constructor(address _traitForgetNft) {	// @audit-issue missing `@param` tag

36:  function setAllowedCaller(address _allowedCaller) external onlyOwner {	// @audit-issue missing `@param` tag

123:  function getPublicEntropy(	// @audit-issue missing `@param` tag

136:  function deriveTokenParameters(	// @audit-issue missing `@param` tag

164:  function getEntropy(	// @audit-issue missing `@param` tag

188:  function numberOfDigits(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@param` tag

198:  function getFirstDigit(uint256 number) private pure returns (uint256) {	// @audit-issue missing `@param` tag
```
[30](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L30-L30), [36](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L36-L36), [123](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L123-L123), [136](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L136-L136), [164](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L164-L164), [188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L188-L188), [198](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L198-L198), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Modifier declarations should have `@notice` tag
In Solidity, the `@notice` tag in NatSpec comments is used to provide important explanations to end users about what a modifier does. It appears that this contract's modifier declarations are missing `@notice` tags in their NatSpec annotations.

The absence of `@notice` tags reduces the contract's transparency and could lead to misunderstandings about a modifier's purpose and behavior.  [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {	// @audit-issue missing `@notice` tag
```
[51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L51-L51), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

25:  modifier onlyAllowedCaller() {	// @audit-issue missing `@notice` tag
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L25-L25), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Modifier declarations should have `@dev` tag
Some modifiers have an incomplete NatSpec: add a `@dev` notation to describe the modifier to improve the code documentation.

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {	// @audit-issue missing `@dev` tag
```
[51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L51-L51), 


```solidity
Path: ./contracts/EntropyGenerator/EntropyGenerator.sol

25:  modifier onlyAllowedCaller() {	// @audit-issue missing `@dev` tag
```
[25](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/EntropyGenerator/EntropyGenerator.sol#L25-L25), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).

### NatSpec: Modifiers `@param` tag is missing
Function modifiers should include NatSpec comments with @param tags describing each input parameter. This promotes better code readability and documentation. [Dive Deeper into NatSpec Guidelines](https://docs.soliditylang.org/en/develop/natspec-format.html)

```solidity
Path: ./contracts/TraitForgeNft/TraitForgeNft.sol

51:  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {	// @audit-issue missing `@param` tag
```
[51](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/./contracts/TraitForgeNft/TraitForgeNft.sol#L51-L51), 


#### Recommendation

Follow the official [Solidity guidelines](https://docs.soliditylang.org/en/v0.8.17/style-guide.html).