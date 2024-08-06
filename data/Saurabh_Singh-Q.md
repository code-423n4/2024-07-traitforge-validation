# L-01
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
The above calculation can result a number with less than 5 digits.
## Impact:-
The comment specifies that `initialNukeFactor` should be a five-digit number. However, the entropy value fetched from TraitForgeNft.sol (calculated as a six-digit number in EntropyGenerator.sol) can result in a value less than five digits when divided by 40, leading to potential calculation errors within the contract.

## Proof Of Concept:-
Example :- Because the value of entropy is a random number lets take a random number of 6 digits [123456]
```
if we divide 123456 by 40
123456/40 will result a number with 4 digits 
```
Numbers with less than 5 digits will trigger further calculation errors within the contract, preventing the project from functioning as intended.

## Tool Used:-
Manual review

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
The incorrect calculation of the `slotIndexSelection` variable within the `EntropyGenerator.sol::initializeAlphaIndices` function creates a bias, unfairly limiting alphaIndex assignments to `entropySlots` array indices greater than 512.

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

```
## Vulnerability Details:-
The calculation within the `EntropyGenerator.sol::initializeAlphaIndices` function is implemented incorrectly, introducing a bias into the system.
```
uint256 slotIndexSelection = (hashValue % 258) + 512;
```
The calculation limits `slotIndexSelection` assignments to indices 512 to 769, excluding lower indices.
## Impact:-
The lower indices in the array will never be assign as alpha index. 

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
## Tool used:-
Manual Review

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

```

# L-03
`EntropyGenerator.sol::initializeAlphaIndices` use a weak random number generator  can be manipulated by user.

## Summary
The random number generator in the `EntropyGenerator.sol::initializeAlphaIndices` is weak and can be manipulated by user to get a number which is profitable to them.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L207
```
function initializeAlphaIndices() public whenNotPaused onlyOwner {
@>        uint256 hashValue = 
            uint256(keccak256(abi.encodePacked(blockhash(block.number - 1), 
                     block.timestamp)));

        uint256 slotIndexSelection = (hashValue % 258) + 512;
        uint256 numberIndexSelection = hashValue % 13;

        slotIndexSelectionPoint = slotIndexSelection; 
        numberIndexSelectionPoint = numberIndexSelection; 
    }
```
## Vulnerability Details:-
The weak random number generator can be manipulated buy the user.
```
uint256 hashValue = 
            uint256(keccak256(abi.encodePacked(blockhash(block.number - 1), 
                     block.timestamp)));
```

The randomness in the EntropyGenerator.sol::initializeAlphaIndices function can be exploited to generate a hash value that points to a specific user index. This allows users to manipulate the system to obtain alpha index values.
## Impact:-
The user can manipulate the hash value and can get alpha index entropy.

## Recommanded Mitigation:-

Integrating Chainlink's Verifiable Random Function (VRF) can provide a robust solution for generating truly random numbers, eliminating the possibility of manipulation and ensuring fairness in the alpha index assignment process.

# L-04

The `EntropyGenerator.sol::writeEntropyBatch3` function does not include a required check for the alpha index rentropy value of 999999.

## Vulnerability Details:-
the `EntropyGenerator.sol::writeEntropyBatch3` do not have a require check for 999999 value of alphaIndex.
```
function writeEntropyBatch3() public {
        require(
            lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,
            "Batch 3 not ready or already completed."
        );
        unchecked {
            for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {
                uint256 pseudoRandomValue = uint256(keccak256(abi.encodePacked(block.number, i))) % uint256(10) ** 78;
                
                entropySlots[i] = pseudoRandomValue;
            }
        }
        lastInitializedIndex = maxSlotIndex;
    }
```
The above-given function should implement a require check to prevent the value 999999 from being populated in the `entropySlots` array.

## Impact:-

since the `entropySlots` array is populated by a random number generator function. the array can get the aplhaIndex value in it.

## Tool used:-
Manual Review

## Recommanded Mitigation:-
```
function writeEntropyBatch3() public {
        require(
            lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,
            "Batch 3 not ready or already completed."
        );
        unchecked {
            for (uint256 i = lastInitializedIndex; i < maxSlotIndex; i++) {
                uint256 pseudoRandomValue = uint256(keccak256(abi.encodePacked(block.number, i))) % uint256(10) ** 78;
+               require(pseudoRandomValue != 999999, "Invalid value, retry.");
                entropySlots[i] = pseudoRandomValue;
            }
        }
        lastInitializedIndex = maxSlotIndex;
    }
```

# L-05
The `EntityForging.sol::listForForging` function has an incorrect implementation for checking the forge potential of a token.

## Vulnerability Details:-
 Entropy is a random 6-digit number and the fifth digit is zero, then the user will be unable to list their token NFT for forging due to a faulty require check implementation.
```
     uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy
    require(
      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
      'Entity has reached its forging limit'
    );
```
Additionally, the condition `forgingCounts[tokenId] <= forgePotential` is incorrectly implemented. While the forgingCounts for a token increment with each forge, the forgePotential is a random value, which can lead to users being able to list their token for forging indefinitely.

```
function listForForging(
    uint256 tokenId,
    uint256 fee
  ) public whenNotPaused nonReentrant {
    Listing memory _listingInfo = listings[listedTokenIds[tokenId]];

    require(!_listingInfo.isListed, 'Token is already listed for forging');
    require(
      nftContract.ownerOf(tokenId) == msg.sender,
      'Caller must own the token'
    );
    require(
      fee >= minimumListFee,
      'Fee should be higher than minimum listing fee'
    );

    _resetForgingCountIfNeeded(tokenId);

    uint256 entropy = nftContract.getTokenEntropy(tokenId); // Retrieve entropy for tokenId
@>    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy
@>    require(
      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
      'Entity has reached its forging limit'
    );

    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy
    require(isForger, 'Only forgers can list for forging');

    ++listingCount;
    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);
    listedTokenIds[tokenId] = listingCount;

    emit ListedForForging(tokenId, fee);
  }
```

## Impact:-
Users with a 0 as the fifth digit in their entropy will be permanently unable to list their tokens for forging, resulting in unfair treatment and a lack of equal opportunity within the game.

## Proof Of code :-
let the value of entropy is 589412.
```
forgePotential = 1 // 5th fifth digit

// the user will be able to list its token only twice because of the following condition in funciton.

require(
      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential
```

## Recommanded Mitigation:-
```
function listForForging(
    uint256 tokenId,
    uint256 fee
  ) public whenNotPaused nonReentrant {
    Listing memory _listingInfo = listings[listedTokenIds[tokenId]];

    require(!_listingInfo.isListed, 'Token is already listed for forging');
    require(
      nftContract.ownerOf(tokenId) == msg.sender,
      'Caller must own the token'
    );
    require(
      fee >= minimumListFee,
      'Fee should be higher than minimum listing fee'
    );

    _resetForgingCountIfNeeded(tokenId);

    uint256 entropy = nftContract.getTokenEntropy(tokenId); // Retrieve entropy for tokenId
-    uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy
-    require(
      forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
      'Entity has reached its forging limit'
    );
+   // we should declare a storage variable at start of contract upto how many time every user can forge.
+   // lets say forgePotential =10.
+    require(forgingCounts[tokenId] <= forgePotential,
      'Entity has reached its forging limit');
+ // This will give every user a fair number of forge.

    bool isForger = (entropy % 3) == 0; // Determine if the token is a forger based on entropy
    require(isForger, 'Only forgers can list for forging');

    ++listingCount;
    listings[listingCount] = Listing(msg.sender, tokenId, true, fee);
    listedTokenIds[tokenId] = listingCount;

    emit ListedForForging(tokenId, fee);
  }
```