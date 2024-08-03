## Title 
Inaccurate age Calculation due to a fixed 365-day year


## Description 

The `NukeFund::calculateAge` function uses a fixed 365-day year for age calculations, without accounting for leap years. This can lead to gradually increasing inaccuracy in age calculations over time.


## Impact 

Using a fixed 365-day year results in gradual drift in age calculations, with the inaccuracy increasing over time, which would be particularly noticeable for older NFTs.

## Proof of Concept 

```
function calculateAge(uint256 tokenId) public view returns (uint256) {
    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');

    uint256 daysOld = (block.timestamp -
      nftContract.getTokenCreationTimestamp(tokenId)) /
      60 /
      60 /
      24;
    uint256 perfomanceFactor = nftContract.getTokenEntropy(tokenId) % 10;

    uint256 age = (daysOld *
      perfomanceFactor *
      MAX_DENOMINATOR *
      ageMultiplier) / 365; // Fixed 365-day year
    return age;
}

```

## Recommendation 

To improve the accuracy of age calculations, consider the following fix 

```
function calculateAge(uint256 tokenId) public view returns (uint256) {
    require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');

    uint256 daysOld = (block.timestamp -
            nftContract.getTokenCreationTimestamp(tokenId)) / 60 /  60 / 24;

    uint256 performanceFactor = nftContract.getTokenEntropy(tokenId) % 10;

    uint256 age = (daysOld *
      performanceFactor *
      MAX_DENOMINATOR *
      ageMultiplier) / 365.25 days; // Use 365.25 days for leap year average
    return age;
}
```

the same issue was found in automated findings in a different contract that isn't imported in NukeFund contract 

https://github.com/code-423n4/2024-07-traitforge/blob/main/4naly3er-report.md#l-14-a-year-is-not-always-365-days