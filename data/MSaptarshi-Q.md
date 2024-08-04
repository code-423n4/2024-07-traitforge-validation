# [QA-1] Contract inherits from `Pausable` but does not expose pausing/unpausing functionality
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L6
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L6
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L7
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L5
The TraitForge smart contracts inherits from OpenZeppelin's Pausable contract, but the _pause and _unpause methods are not exposed externally to be callable This shows that Pausable was used incorrectly and is possible to give out a false sense of security when actually contract is not pausable at all.

## Recommendations
expose the _pause and _unpause methods externally with access control.


# [QA-2] Minting doesn't check if the NFT exists or not
Important functions like minting a NFT should check a NFT exists or not in the 1st place, Since minting the same NFT twice would result in duplicate NFT's 
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L181
```
function mintToken(
    bytes32[] calldata proof
  )
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
  {
    uint256 mintPrice = calculateMintPrice();
    require(msg.value >= mintPrice, 'Insufficient ETH send for minting.');

    _mintInternal(msg.sender, mintPrice);

    uint256 excessPayment = msg.value - mintPrice;
    if (excessPayment > 0) {
      (bool refundSuccess, ) = msg.sender.call{ value: excessPayment }('');
      require(refundSuccess, 'Refund of excess payment failed.');
    }
  }
```
## Recommendation
Before minting a NFT check if the `tokenId` exists