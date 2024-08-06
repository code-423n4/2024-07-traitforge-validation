# Findings Summary

| ID     | Title                                                                | Severity     |
| ------ | -------------------------------------------------------------------- | --------     |
| [L-01] | Arbitrageurs can always mint high-value NFTs without loss            | Low          |
| [L-02] | NFT.mint lack slippage protection                                    | Low          |
| [L-03] | ForgeWithListed did not refund excess funds                          | Low          |
| [L-04] | Entropy may be zero causing loss of user funds                       | Low          |
| [N-01] | There seems to be a problem with the calculation of slotValue        | Non-Critical |

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

# [N-01] There seems to be a problem with the calculation of slotValue

## Description

```solidity
    function writeEntropyBatch1() public {
        require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');

        uint256 endIndex = lastInitializedIndex + batchSize1; // calculate the end index for the batch
        unchecked {
            for (uint256 i = lastInitializedIndex; i < endIndex; i++) {
                uint256 pseudoRandomValue = uint256(
                    keccak256(abi.encodePacked(block.number, i))
                ) % uint256(10) ** 78; // generate a  pseudo-random value using block number and index
                require(pseudoRandomValue != 999999, 'Invalid value, retry.');
                entropySlots[i] = pseudoRandomValue; // store the value in the slots array
            }
        }
        lastInitializedIndex = endIndex;
    }

```

According to the whitepaper, each slot contains a 78 digit number, but the code implementation performs a modulo operation on `uint256(10) ** 78`, which seems to be an incorrect implementation.
Because `type(uint256).max < uint256(10) ** 78`, `uint256(10) ** 78` will only overflow to a value within uint256, causing slotValue to be smaller than the overflowed value, which is a 77 digit number.

## Recommendations

There is no need to take the remainder here.
