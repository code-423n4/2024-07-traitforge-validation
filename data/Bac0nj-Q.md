# List for forging can be overpassed by witch attacks
```solidity
// EntityForging.sol: 111
    require(
      nftContract.ownerOf(mergerTokenId) == msg.sender,
      'Caller must own the merger token'
    );
    require(
      nftContract.ownerOf(forgerTokenId) != msg.sender,
      'Caller should be different from forger token owner'
    );
```
forgeWithListed checks and make sure the owner can't forge with its own forger. If the owner uses multiple addresses, then the check was made pointless.