### [L-1] `TraitForgeNft::isForge()` will always return true if NFT with tokenId is not minted yet (Root + Impact)

**Description**

The role indicator is determined by taking the modulo 3 of the token's entropy. This entropy value is stored in a mapping within the contract. If the NFT with the specified `tokenId` has not been minted yet, the entropy value in the mapping will be `0`. Since `0 % 3 = 0`, this condition will always evaluate to true.

**Impact**

This function, if used by the front-end or users, could provide incorrect information.

**Proof of Concepts**

```javascript
function isForger(uint256 tokenId) public view returns (bool) {
@>    uint256 entropy = tokenEntropy[tokenId]; 
@>    uint256 roleIndicator = entropy % 3;
      return roleIndicator == 0;
}
```

**Recommended mitigation**

```diff
function isForger(uint256 tokenId) public view returns (bool) {
+   require(_exists(tokenId), "ERC721: operator query for nonexistent token");
    uint256 entropy = tokenEntropy[tokenId]; //!If the token is not minted yet, it will always return true;
    uint256 roleIndicator = entropy % 3;
    return roleIndicator == 0;
}
```

### [L-2] Dev cut is not entirely distributed to devs
**Description**

In the [DevFund::receive()](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L14) function, where rewards are distributed among developers, any loss of precision is sent to the contract owner, thereby reducing the rewards for the developers.

```javascript
receive() external payable {
    if (totalDevWeight > 0) {
        uint256 amountPerWeight = msg.value / totalDevWeight;
@>      uint256 remaining = msg.value - (amountPerWeight * totalDevWeight);
        totalRewardDebt += amountPerWeight;
        if (remaining > 0) {
@>          (bool success,) = payable(owner()).call{value: remaining}("");
            require(success, "Failed to send Ether to owner");
        }
    } else {
        (bool success,) = payable(owner()).call{value: msg.value}("");
        require(success, "Failed to send Ether to owner");
    }
    emit FundReceived(msg.sender, msg.value);
}

```

**Impact**

In the case of numerous trades, minting, and forging, these small amounts could accumulate significantly over time.

**Recommended mitigation**

A constant could be used to handle floating-point precision.
