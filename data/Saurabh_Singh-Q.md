# L-01:
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
# L-02
The `slotIndexSelection` variable within the `EntropyGenerator.sol::initializeAlphaIndices` function introduces a bias, limiting the possibility of receiving an alphaIndex to users whose entropy values originate from `entropySlots` array indices 512 and above.

## Summary:-
The incorrect calculation of the slotIndexSelection variable within the EntropyGenerator.sol::initializeAlphaIndices function creates a bias, unfairly limiting alphaIndex assignments to entropySlots array indices greater than 512.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L211

```
  //select index points for 999999, triggered each gen-increment
  function initializeAlphaIndices() public whenNotPaused onlyOwner {
    uint256 hashValue = uint256(
      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))
    );

@>    uint256 slotIndexSelection = (hashValue % 258) + 512;
      uint256 numberIndexSelection = hashValue % 13;

    slotIndexSelectionPoint = slotIndexSelection;
    numberIndexSelectionPoint = numberIndexSelection;
  }
}
```
## Vulnerability Details:-
The calculation within the EntropyGenerator.sol::initializeAlphaIndices function is implemented incorrectly, introducing a bias into the system.
```
uint256 slotIndexSelection = (hashValue % 258) + 512;
```
The calculation limits `slotIndexSelection` assignments to indices 512 to 769, excluding lower indices.

## Proof of code :-
The calculation given 
lets assume the hashValue a random number e.g[8462213]
```
uint256 slotIndexSelection = (hashValue % 258) + 512;

(8462213 %258) +512;
71+512
583.

```
if the modulus part return 0 for any other number then
0+512 =512  there is no chance for lower indices.

## Recommanded Mitigation:-
Remove the old calculation and add the line as given below which return indices from 0 to 769.
```
  //select index points for 999999, triggered each gen-increment
  function initializeAlphaIndices() public whenNotPaused onlyOwner {
    uint256 hashValue = uint256(
      keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp))
    );

-     uint256 slotIndexSelection = (hashValue % 258) + 512;
+     uint256 slotIndexSelection = (hashValue % 770);
      uint256 numberIndexSelection = hashValue % 13;

    slotIndexSelectionPoint = slotIndexSelection;
    numberIndexSelectionPoint = numberIndexSelection;
  }
}
```