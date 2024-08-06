### 1. The ability and responsibility of the owner to control all protocol variables increases the chance of the protocol being broken.

Owner controll almost all variables, most of them must be initialized as soon as possible after deployment of the protocol. Otherwise protocol will be broken, more over, whitelist time window (24 hours) will be wasted if one or more variables uninitialized.

Owner must call all this functions for protocol to work, if any missing whitelisted users waste whitelist time
```solidity
nft.setEntityForgingContract(address(entityForging));
nft.setEntropyGenerator(address(entropyGenerator));
nft.setAirdropContract(address(airdrop));
airdrop.setTraitToken(address(token));
airdrop.transferOwnership(address(nft));
nft.setNukeFundContract(payable(address(nukeFund)));

entropyGenerator.transferOwnership(address(nft));

entityTrading.setNukeFundAddress(payable(address(nukeFund)));
entityForging.setNukeFundAddress(payable(address(nukeFund)));

entropyGenerator.writeEntropyBatch1();
entropyGenerator.writeEntropyBatch2();
entropyGenerator.writeEntropyBatch3();
```

As a solution, either use standard initializer pattern or make all important variables that required to protocol works correctly to update `whitelistEndTime` if it is not happend already

### 2. Memory array in `EntityForging.fetchListings` in created with wrong length

There is no need in `listingCount + 1`. Just `listingCount` would fit.

[EntityForging:48-53](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L48-L53)

```solidity
function fetchListings() external view returns (Listing[] memory _listings) {
    _listings = new Listing[](listingCount + 1); // @audit-qa array is bigger than needed
    for (uint256 i = 1; i <= listingCount; ++i) {
        _listings[i] = listings[i];
    }
}
```

### 3. Deployer will lost a lot of gas if there `999999` in entropy generation

Currently `writeEntropyBatch1` and `writeEntropyBatch2` revert if `pseudoRandomValue == 999999` Which cause part of 15M gas transaction to waste gas for nothing.
It is better to reroll and generate new value if `pseudoRandomValue` == 999999.

[EntropyGenerator:56](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L56), [EntropyGenerator:76](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L76)

```solidity
uint256 pseudoRandomValue = uint256(keccak256(abi.encodePacked(block.number, i))) % uint256(10) ** 78;
require(pseudoRandomValue != 999999, "Invalid value, retry.");
entropySlots[i] = pseudoRandomValue;
```

### 4. `mintToken` should return minted tokenId

Currently mintToken, return nothing and user have to listen to events to get its tokenId number

[TraitForgeNft:181-188](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L181-L188)

```solidity
function mintToken(bytes32[] calldata proof)
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
{
```

### 5. `EntropyGenerator.deriveTokenParameters` Has completely wrong logic
Logic for `nukeFactor` and `forgePotential` is wrong, but function is not used in rest of protocol, so can't be critical

[EntropyGenerator:136-161](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L136-L161)

```solidity
function deriveTokenParameters(uint256 slotIndex, uint256 numberIndex)
    public
    view
    returns (uint256 nukeFactor, uint256 forgePotential, uint256 performanceFactor, bool isForger)
{
    uint256 entropy = getEntropy(slotIndex, numberIndex);

    // example calcualtions using entropyto derive game-related parameters
    nukeFactor = entropy / 4000000; // @audit-high Always ZERO???
    forgePotential = getFirstDigit(entropy); // @audit-high The wrong logic 
    performanceFactor = entropy % 10;

    // exmaple logic to determine a boolean property based on entropy
    uint256 role = entropy % 3;
    isForger = role == 0;

    return (nukeFactor, forgePotential, performanceFactor, isForger); // return derived parammeters
}
```

### 6. User can be frontrunned, leaving him with worse entropy

The user can be frontruned, which will lead to a worsening of his entropy. There are no means of protection against frontrunning, although the entropy is publicly known, and anyone can frontrun an honest user to take his token with better entropy.

