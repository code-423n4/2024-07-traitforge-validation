1. Inconsistent token existence checks leading to false positives

<h2>Affected lines of code<h2>

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L274-L278

<h2>Impact<h2>
The function will incorrectly return a bool for deleted tokens.

<h2>Proof of concept<h2>
The isForger function does not include a token existence check before accessing the tokenEntropy mapping. 

```
function isForger(uint256 tokenId) public view returns (bool) {
    uint256 entropy = tokenEntropy[tokenId];
    uint256 roleIndicator = entropy % 3;
    return roleIndicator == 0;
}

```

As a result, when queried with a tokenId that has been deleted through calling the nuke(), the function will return true instead of reverting.

Note that in the _mintInternal(), token entropy for the minted token is stored in tokenEntropy mapping. 

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L291

So when a token is nuked and the id of the token nuked is called in the isForger(), it doesn't revert.

<h2>Recommendation<h2>
Modify the isForger function to include a token existence check:

function isForger(uint256 tokenId) public view returns (bool) {
    require(ownerOf(tokenId) != address(0), "ERC721: query for nonexistent token");
    uint256 entropy = tokenEntropy[tokenId];
    uint256 roleIndicator = entropy % 3;
    return roleIndicator == 0;
}




