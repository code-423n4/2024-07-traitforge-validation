# Findings Summary

| ID     | Title                                                                | Severity |
| ------ | -------------------------------------------------------------------- | -------- |
| [L-01] | Arbitrageurs can always mint high-value NFTs without loss            | Low      |
| [L-02] | NFT.mint lack slippage protection                                    | Low      |
| [L-03] | ForgeWithListed did not refund excess funds                          | Low      |
| [L-04] | Entropy may be zero causing loss of user funds                       | Low      |

# Detailed Findings

# [L-01] Arbitrageurs can always mint high-value NFTs without loss

## Description

```solidity
// Calculate the nuke factor of a token, which affects the claimable amount from the fund
function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {
    require(
        nftContract.ownerOf(tokenId) != address(0),
        'ERC721: operator query for nonexistent token'
    );

    uint256 entropy = nftContract.getTokenEntropy(tokenId);
    uint256 adjustedAge = calculateAge(tokenId);

    uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits

    uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) /
        MAX_DENOMINATOR) + initialNukeFactor;

    return finalNukeFactor;
}
```

The value of NFT depends on entropy and creation time, and entropy can be calculated off-chain or on-chain.      
Therefore, arbitrageurs only need to judge the value of NFT at the time of minting to decide whether to revert the minted transaction to achieve lossless arbitrage.      
Since the cost of the Base chain is not taken into account, the arbitrage cost is very low.     

## Recommendations

Use random numbers instead of precomputed data to calculate entropy.

# [L-02] NFT.mint lack slippage protection

## Description

```solidity
    function calculateMintPrice() public view returns (uint256) {
        uint256 currentGenMintCount = generationMintCounts[currentGeneration];
        uint256 priceIncrease = priceIncrement * currentGenMintCount;
        uint256 price = startPrice + priceIncrease;
        return price;
    }
```

The price of NFT increases gradually with the number of minted coins. The user's transaction may be delayed, resulting in the minting cost exceeding the user's expectations. In particular, `mintWithBudget` allows users to mint in batches, which may cause financial losses.  

## Recommendations

The `mintWithBudget` function should add slippage protection for single minting price

# [L-03] ForgeWithListed did not refund excess funds

## Description

```solidity
    uint256 forgingFee = _forgerListingInfo.fee;
    require(msg.value >= forgingFee, 'Insufficient fee for forging');
```

ForgeWithListed allows users to pass in excess funds, but there is no refund, which may cause some funds to be stuck in the contract forever.       
Other contracts refunds excess funds, and consistency should be maintained here as well.     

## Recommendations

ForgeWithListed should refund excess funds

# [L-04] Entropy may be zero causing loss of user funds

## Description

```solidity
    // private function to calculate the entropy value based on slot and number index
    function getEntropy(
        uint256 slotIndex,
        uint256 numberIndex
    ) private view returns (uint256) {
        require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');

        if (
            slotIndex == slotIndexSelectionPoint &&
            numberIndex == numberIndexSelectionPoint
        ) {
            return 999999;
        }

        uint256 position = numberIndex * 6; // calculate the position for slicing the entropy value
        require(position <= 72, 'Position calculation error');

        uint256 slotValue = entropySlots[slotIndex]; // slice the required [art of the entropy value
        uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000; // adjust the entropy value based on the number of digits
        uint256 paddedEntropy = entropy * (10 ** (6 - numberOfDigits(entropy)));

        return paddedEntropy; // return the caculated entropy value
    }
```

Allowing users to mint NFTs before calling writeEntropyBatch, with an entropy value of 0 and the value of nft is 0, will cause users to lose funds.        
In addition, the entropy calculation formula may also produce a zero value when there are multiple zeros at the end of slotValue.       

## Recommendations

Should not allow users to mint nft if entropy is zero.
