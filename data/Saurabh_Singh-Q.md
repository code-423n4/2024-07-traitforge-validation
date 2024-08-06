The `initialNukeFactor` variable in the `NukeFund.sol::calculateNukeFactor` function is calculated incorrectly, resulting in a wrong value.

## Summary:-
The calculation for `initialNukeFactor` within the `NukeFund.sol::calculateNukeFactor` function is incorrect.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L145

```
    function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {
        require(nftContract.ownerOf(tokenId) != address(0), "ERC721: operator query for nonexistent token");

        uint256 entropy = nftContract.getTokenEntropy(tokenId);
        uint256 adjustedAge = calculateAge(tokenId);

@>      uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits
        uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) / MAX_DENOMINATOR) + initialNukeFactor;

        return finalNukeFactor;
    }
```

## Vulnerability Details:-
The variable `initialNukeFactor` is calculated as 
```
        uint256 entropy = nftContract.getTokenEntropy(tokenId);
        uint256 adjustedAge = calculateAge(tokenId);

@>      uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits
```

The comment specifies that initialNukeFactor should be a five-digit number. However, the entropy value fetched from TraitForgeNft.sol (calculated as a six-digit number in EntropyGenerator.sol) can result in a value less than five digits when divided by 40, leading to potential calculation errors within the contract.

## Proof Of Concept:-
Example :- Because the value of entropy is a random number lets take a random number of 6 digits [123456]
```
if we divide 123456 by 40
123456/40 will result a number with 4 digits 
```
Numbers with less than 5 digits will trigger further calculation errors within the contract, preventing the project from functioning as intended.

## Recommanded Mitigation:-

To correct this bug and restore the contract's intended behavior, the following actions will be implemented.

```
function calculateNukeFactor(uint256 tokenId) public view returns (uint256) {
        require(nftContract.ownerOf(tokenId) != address(0), "ERC721: operator query for nonexistent token");

        uint256 entropy = nftContract.getTokenEntropy(tokenId);
        uint256 adjustedAge = calculateAge(tokenId);

-      uint256 initialNukeFactor = entropy / 40; // calcualte initalNukeFactor based on entropy, 5 digits
+      uint256 initialNukeFactor = entropy / 10;
        uint256 finalNukeFactor = ((adjustedAge * defaultNukeFactorIncrease) / MAX_DENOMINATOR) + initialNukeFactor;

        return finalNukeFactor;
    }
```
