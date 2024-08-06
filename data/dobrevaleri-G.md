## Gas Optimization

### [GO-1] Unused variable
**Description**

The `amountMinted` in the [TraitForgeNft::mintWithBudget()](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L202) is not used anywhere.

```javascript
function mintWithBudget(bytes32[] calldata proof)
        public
        payable
        whenNotPaused
        nonReentrant
        onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
    {
        uint256 mintPrice = calculateMintPrice();
@>      uint256 amountMinted = 0;
        uint256 budgetLeft = msg.value;

        while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
            _mintInternal(msg.sender, mintPrice);
@>          amountMinted++;
            budgetLeft -= mintPrice;
            mintPrice = calculateMintPrice();
        }
        if (budgetLeft > 0) {
            (bool refundSuccess,) = msg.sender.call{value: budgetLeft}("");
            require(refundSuccess, "Refund failed.");
        }
    }
```
**Recommended mitigation**

```diff
function mintWithBudget(bytes32[] calldata proof)
        public
        payable
        whenNotPaused
        nonReentrant
        onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
    {
        uint256 mintPrice = calculateMintPrice();
-       uint256 amountMinted = 0;
        uint256 budgetLeft = msg.value;

        while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
            _mintInternal(msg.sender, mintPrice);
-           amountMinted++;
            budgetLeft -= mintPrice;
            mintPrice = calculateMintPrice();
        }
        if (budgetLeft > 0) {
            (bool refundSuccess,) = msg.sender.call{value: budgetLeft}("");
            require(refundSuccess, "Refund failed.");
        }
    }
```
