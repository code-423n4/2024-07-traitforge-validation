[L01]improper approval check in nuke()
In nuke() function, we have two require check. The first require check is we need to make sure the msg.sender has the permission to process the NFT token.
And the second require is that we need to make sure the NFT owner has approved `NukeFund` contract to process this NFT. Because `NukeFund` contract will burn NFT owner's NFT.
The problem exists in the second require. Considering NFT owner has approved `NukeFund` and when one operator of this NFT try to nuke, this operation might be reverted.
`nftContract.isApprovedForAll(msg.sender, address(this))`, the `msg.sender` should be changed to the owner of NFT.
```javascript
  function nuke(uint256 tokenId) public whenNotPaused nonReentrant {
    // Check the permission
    require(
      nftContract.isApprovedOrOwner(msg.sender, tokenId),
      'ERC721: caller is not token owner or approved'
    );
    require(
      nftContract.getApproved(tokenId) == address(this) ||
@=>   nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );
    ...
}
```