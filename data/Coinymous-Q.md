# [QA] Inefficient Fund Distribution in `TraitForgeNft::mintWithBudget` Leads to Increased Gas Usage

## Description
In the `TraitForgeNft::mintWithBudget` function, each NFT minting operation triggers the `TraitForgeNft::_distributeFunds` function to transfer the corresponding received fee. This process is repeated for every NFT minted, leading to multiple fund transfers that increase gas usage and the probability of reaching the block gas limit during multiple token mints.

## Recommended Mitigation Steps
To optimize gas usage, accumulate the total funds received and call `TraitForgeNft::_distributeFunds` only once at the end of the minting process. This change reduces the number of fund transfer operations, thereby decreasing gas usage.

### Detailed Mitigation Steps
1. **Remove `_distributeFunds` from `_mintInternal` function:**
    ```diff
    File: ./contracts/TraitForgeNft/TraitForgeNft.sol::_mintInternal

    - 308       _distributeFunds(mintPrice);
    ```

2. **Add `_distributeFunds` to both `mintToken` and `mintWithBudget` functions:**
    ```diff
    File: ./contracts/TraitForgeNft/TraitForgeNft.sol::mintToken

    193      _mintInternal(msg.sender, mintPrice);
    +        _distributeFunds(mintPrice);
    ```

    ```diff
    File: ./contracts/TraitForgeNft/TraitForgeNft.sol::mintWithBudget

    215   while (budgetLeft >= mintPrice && _tokenIds < maxTokensPerGen) {
              _mintInternal(msg.sender, mintPrice);
              amountMinted++;
              budgetLeft -= mintPrice;
              mintPrice = calculateMintPrice();
          }
    +     _distributeFunds(msg.value - budgetLeft);
          if (budgetLeft > 0) {
              (bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
              require(refundSuccess, 'Refund failed.');
          }
    ```

By implementing these changes, the `TraitForgeNft::mintWithBudget` function will call `_distributeFunds` only once, reducing the overall gas usage and ensuring efficient fund transfer operations. This approach maintains the functionality while optimizing gas consumption, thereby improving the performance of the contract.