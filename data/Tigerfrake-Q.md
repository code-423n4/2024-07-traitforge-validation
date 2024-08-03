### [L-1] Refactor `DevFund::claim()` to eliminate redundancy
The [DevFund::claim()](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L64-L66) function performs operations all over again which are already configured in a single function:
```solidity
    function claim() external whenNotPaused nonReentrant {
        DevInfo storage info = devInfo[msg.sender];

>>      uint256 pending = info.pendingRewards +
        (totalRewardDebt - info.rewardDebt) *
        info.weight;
        //..
    }
```
However, the function can be refactored to call the existing function [`pendingRewards()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L77-L81) to avoid redundant.

**Recommendation**
Refactor as follows
```diff
    function claim() external whenNotPaused nonReentrant {
        DevInfo storage info = devInfo[msg.sender];

-       uint256 pending = info.pendingRewards +
-       (totalRewardDebt - info.rewardDebt) *
-       info.weight;
+       uint256 pending = pendingRewards(msg.sender);
        //..
    }
```

### [L-2] Inaccurate error message thrown in `listForForging()`
The [`listForForging()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L78-L81) function throws an error message that is not accurate. 
```solidity
    require(
      fee >= minimumListFee,
      'Fee should be higher than minimum listing fee' //@audit-info Incorrect error message
    );
```
The error message `'Fee should be higher than minimum listing fee'` is thrown when in reality the function also allows `fee` equal to `minimumListFee`.

**Recommendation**
Correct this as follows:
```diff
    require(
      fee >= minimumListFee,
-     'Fee should be higher than minimum listing fee'
+     'Fee should be higher than or equal to minimum listing fee'
    );
```

### [L-3] Use Correct and secure version of `transferToNukeFund` function in all cases
In most the contracts, the `nukeFundAddress` is not initialized. However there exist a setter function for this `setNukeAddress()`. The issue here is that there could exist exist a window before this addres is set by the owner meaning that the `nukeFundAddress` will be `address(0)` during this time. 
This could lead to burning of funds.

Consider the following:
1. [`EntityForging::forgeWithListed()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L156-L157)
```solidity
(bool success, ) = nukeFundAddress.call{ value: devFee }('');
require(success, 'Failed to send to NukeFund');
```

2. [`TraitForgeNft::_distributeFunds()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L361-L362)
```solidity
(bool success, ) = nukeFundAddress.call{ value: totalAmount }('');
require(success, 'ETH send failed');
```

#### Recommendation
Make a call to [`EntityTrading::transferToNukeFund()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L111-L117) to perform the transfer safely as shown here:
```solidity
  // Correct and secure version of transferToNukeFund function
  function transferToNukeFund(uint256 amount) private {
    require(nukeFundAddress != address(0), 'NukeFund address not set');
    (bool success, ) = nukeFundAddress.call{ value: amount }('');
    require(success, 'Failed to send Ether to NukeFund');
    emit NukeFundContribution(address(this), amount);
  }
```

### [L-4] Invoke `isForger()` rather than creating new implemetation:
The [`listForForging()`]() function can be refactored to eliminate redundancy and improve readability.
```solidity
bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy
```

#### Recommendation
Instead of rewriting the implemetation to check if a user `isforger`, just invoke [`TraitForgeNft::isForger()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L274-L278):
```diff
-   bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy
+   bool isForger = nftContract.isForger(tokenId);
```

### [L-5] Unlimited Permissions on Token Approval
**Description**:
Unlimited Approval is a feature that enables users to grant platforms and smart contracts the permission to spend tokens/ coins on your behalf without limit. Normally, when swapping on a specific AMM such as Uniswap, users need to approve the smart contract/ platform to transfer those tokens on their behalf.

This is also done within this protocol in functions such as [`listNFTForSale()`](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L47-L51):
```solidity
    require(
      nftContract.getApproved(tokenId) == address(this) ||
>>      nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );
```
When giving these platforms/ smart contracts unlimited approval, users will face 2 main risks: malicious projects and bug exploits. This risk gives hackers the right to use the approved tokens for many lucrative purposes.

*References*:
https://metamask.zendesk.com/hc/en-us/articles/6174898326683-What-is-a-token-approval-#h_01G6X0J818RMX8E35CCPE0KQEH

**Recommendation**
Be sure to revoke these approvals if the user calls [`cancelListing()](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L94-L109) to cancel their NFT from listing.


### [L-6] `merkleProof` length not checked
**Description**:
`proof` length is not checked: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L51-L59
```solidity
  modifier onlyWhitelisted(bytes32[] calldata proof, bytes32 leaf) {
    if (block.timestamp <= whitelistEndTime) {
      require(
        MerkleProof.verify(proof, rootHash, leaf),
        'Not whitelisted user'
      );
    }
    _;
  }
```
**Recommendation**
I recommend adding a check that `proof.length > 0`.
