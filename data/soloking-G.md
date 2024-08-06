# **Title**
### Potential Gas Optimization Issue in mintWithBudget Function

# Links to Affected Code

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L202

# Vulnerability Details
## Impact
If the budget provided is very high, the loop in the `mintWithBudget` function might run into the block gas limit, causing the transaction to fail. This can result in wasted gas fees for the user and unsuccessful minting. Additionally, it can impact the user experience by making the minting process less reliable.

# Proof of Concept
Consider a scenario where the budget provided is significantly higher than the mint price, causing the loop to run many iterations:

```
uint256 mintPrice = calculateMintPrice();
uint256 amountMinted = 0;
uint256 budgetLeft = msg.value;

while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
    _mintInternal(msg.sender, mintPrice);
    amountMinted++;
    budgetLeft -= mintPrice;
    mintPrice = calculateMintPrice();
}

```
In cases where `budgetLeft` is much larger than `mintPrice`, the loop may execute many times, potentially exceeding the block gas limit and causing the transaction to revert.

# Recommended Mitigation Steps
Implement gas limit checks or a cap on the number of tokens that can be minted in a single transaction to prevent exceeding the block gas limit. Here's an updated version of the `mintWithBudget` function with a cap on the number of tokens that can be minted in a single transaction:

## Updated mintWithBudget Function:

```
function mintWithBudget(
    bytes32[] calldata proof
)
    public
    payable
    whenNotPaused
    nonReentrant
    onlyWhitelisted(proof, keccak256(abi.encodePacked(msg.sender)))
{
    uint256 mintPrice = calculateMintPrice();
    uint256 amountMinted = 0;
    uint256 budgetLeft = msg.value;
    uint256 maxMintPerTx = 10; // Set a reasonable cap on the number of tokens per transaction

    while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen && amountMinted < maxMintPerTx) {
        _mintInternal(msg.sender, mintPrice);
        amountMinted++;
        budgetLeft -= mintPrice;
        mintPrice = calculateMintPrice();
    }
    if (budgetLeft > 0) {
        (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
        require(refundSuccess, 'Refund failed.');
    }
}

```
This modification sets a cap on the number of tokens that can be minted in a single transaction, preventing the loop from running too many iterations and potentially exceeding the block gas limit.