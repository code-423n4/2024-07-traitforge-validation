
# [G-01] Using custom errors instead of revert error strings

## Description: 
Using custom errors instead of revert error strings to reduce deployment and runtime cost , Custom errors are available from solidity version 0.8.4. The instances below match or exceed that version

There are 71 instances of this issue:



```
File: 2024-07-traitforge/contracts/DevFund/DevFund.sol
21:  require(success, 'Failed to send Ether to owner');

25:  require(success, 'Failed to send Ether to owner');

32:  require(weight > 0, 'Invalid weight');

33:  require(info.weight == 0, 'Already registered');

42:  require(weight > 0, 'Invalid weight');

43:  require(info.weight > 0, 'Not dev address');

53:  require(info.weight > 0, 'Not dev address');

90:  require(success, 'Failed to send Reward');

```

```
File: 2024-07-traitforge/contracts/EntityForging/EntityForging.sol

73:      require(!_listingInfo.isListed, 'Token is already listed for forging');

74:    require(
        nftContract.ownerOf(tokenId) == msg.sender,
        'Caller must own the token'
77:    );

78:    require(
        fee >= minimumListFee,
        'Fee should be higher than minimum listing fee'
81:    );

87:    require(
        forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
        'Entity has reached its forging limit'
90:    );

93:    require(isForger, 'Only forgers can list for forging');

111:   require(
        nftContract.ownerOf(mergerTokenId) == msg.sender,
        'Caller must own the merger token'
113:  );

115:   require(
        nftContract.ownerOf(forgerTokenId) != msg.sender,
        'Caller should be different from forger token owner'
118:  );

119:  require(    
nftContract.getTokenGeneration(mergerTokenId) ==
        nftContract.getTokenGeneration(forgerTokenId),
      'Invalid token generation'
123:  );

126:    require(msg.value >= forgingFee, 'Insufficient fee for forging');

137:    require(mergerEntropy % 3 != 0, 'Not merger');

140:    require(
          mergerForgePotential > 0 &&
        forgingCounts[mergerTokenId] <= mergerForgePotential,
          'forgePotential insufficient'
144:    );

157:    require(success, 'Failed to send to NukeFund');

159:    require(success_forge, 'Failed to send to Forge Owner');

180:    require(
          nftContract.ownerOf(tokenId) == msg.sender ||
        msg.sender == address(nftContract),
         'Caller must own the token'
184:    );

185:    require(
   listings[listedTokenIds[tokenId]].isListed,
         'Token not listed for forging'
188:    );

```

```
File: 2024-07-traitforge/contracts/EntityTrading/EntityTrading.sol

42:     require(price > 0, 'Price must be greater than zero');

43:     require(
      nftContract.ownerOf(tokenId) == msg.sender,
         'Sender must be the NFT owner.'
46:    );

47:     require(
         nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
        'Contract must be approved to transfer the NFT.'
51:     );

65:     require(
      msg.value == listing.price,
         'ETH sent does not match the listing price'
68:     );

69:     require(listing.seller != address(0), 'NFT is not listed for sale.');


80:     require(success, 'Failed to send to seller');


98:     require(
      listing.seller == msg.sender,
      'Only the seller can canel the listing.'
101:     );


102:     require(listing.isActive, 'Listing is not active.');

113:     require(nukeFundAddress != address(0), 'NukeFund address not set');

115:     require(success, 'Failed to send Ether to NukeFund');
 
```
```
File: 2024-07-traitforge/contracts/EntropyGenerator/EntropyGenerator.sol

26:     require(msg.sender == allowedCaller, 'Caller is not allowed');

48:     require(lastInitializedIndex < batchSize1, 'Batch 1 already initialized.');

56:         require(pseudoRandomValue != 999999, 'Invalid value, retry.');

65:     require(
      lastInitializedIndex >= batchSize1 && lastInitializedIndex < batchSize2,
        'Batch 2 not ready or already initialized.'
68:     );
 
76:         require(pseudoRandomValue != 999999, 'Invalid value, retry.');

85:     require(
      lastInitializedIndex >= batchSize2 && lastInitializedIndex < maxSlotIndex,
         'Batch 3 not ready or already completed.'
88:    );
 
102:     require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');

168:     require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');

178:     require(position <= 72, 'Position calculation error');



```

```
2024-07-traitforge/contracts/NukeFund/NukeFund.sol

48:      require(success, 'ETH send failed');

52:      require(success, 'ETH send failed');

55:      require(success, 'ETH send failed');

119:     require(nftContract.ownerOf(tokenId) != address(0), 'Token does not exist');

137:     require(
            nftContract.ownerOf(tokenId) != address(0),
            'ERC721: operator query for nonexistent token'
140:      );

154:     require(
      nftContract.isApprovedOrOwner(msg.sender, tokenId),
         'ERC721: caller is not token owner or approved'
157:    );

158:    require(
          nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
          'Contract must be approved to transfer the NFT.'
162:  );

163:    require(canTokenBeNuked(tokenId), 'Token is not mature yet');

178:    require(success, 'Failed to send Ether');
 
186:     require(
          nftContract.ownerOf(tokenId) != address(0),
          'ERC721: operator query for nonexistent token'
189:     );
 
```

```
File: 2024-07-traitforge/contracts/TraitForgeNft/TraitForgeNft.sol


53:      require(
           MerkleProof.verify(proof, rootHash, leaf),
           'Not whitelisted user'
56:       );

77:     require(entityForgingAddress_ != address(0), 'Invalid address');

85:     require(entropyGeneratorAddress_ != address(0), 'Invalid address');

91:     require(airdrop_ != address(0), 'Invalid address');

115:    require(
         maxGeneration_ >= currentGeneration,
         "can't below than current generation"
118:    );

142:    require(
         isApprovedOrOwner(msg.sender, tokenId),
         'ERC721: caller is not token owner or approved'
145:   );

159:     require(
          msg.sender == address(entityForgingContract),
         'unauthorized caller'
162:     );

166:     require(newGeneration <= maxGeneration, "can't be over max generation");

191:     require(msg.value >= mintPrice, 'Insufficient ETH send for minting.');

198:     require(refundSuccess, 'Refund of excess payment failed.');

223:     require(refundSuccess, 'Refund failed.');

235:      require(
          ownerOf(tokenId) != address(0),
          'ERC721: query for nonexistent token'
238:      );

257:     require(
          ownerOf(tokenId) != address(0),
          'ERC721: query for nonexistent token'
260:     );

267:     require(
           ownerOf(tokenId) != address(0),
          'ERC721: query for nonexistent token'
270:     );

316:     require(
          generationMintCounts[gen] < maxTokensPerGen,
         'Exceeds maxTokensPerGen'
319:     );

346:     require(
       generationMintCounts[currentGeneration] >= maxTokensPerGen,
          'Generation limit not yet reached'
349:    );


359:     require(address(this).balance >= totalAmount, 'Insufficient balance');

362:     require(success, 'ETH send failed');

394:     require(!paused(), 'ERC721Pausable: token transfer while paused');

```




















# [G-01]**Use `x != 0` instead of `x > 0` for uint types**

## Description: 
The `!=` operator costs less gas than `>` and for uint types you can use it to check for non-zero values to save gas

```
File: 2024-07-traitforge/contracts/DevFund/DevFund.sol

15:    if (totalDevWeight > 0)

19:    if (remaining > 0) 

42:     require(weight > 0, 'Invalid weight');

43:     require(info.weight > 0, 'Not dev address');


53:     require(info.weight > 0, 'Not dev address');


68:     if (pending > 0) {


```

```
File: 2024-07-traitforge/contracts/EntityForging/EntityForging.sol

87:     require(
         forgePotential > 0 && forgingCounts[tokenId] <= forgePotential,
         'Entity has reached its forging limit'
90:     );


140:     require(
          mergerForgePotential > 0 &&
        forgingCounts[mergerTokenId] <= mergerForgePotential,
          'forgePotential insufficient'
144:    );
```



```
File: 2024-07-traitforge/contracts/EntityTrading/EntityTrading.sol

42:    require(price > 0, 'Price must be greater than zero');


```


```
File: 2024-07-traitforge/contracts/TraitForgeNft/TraitForgeNft.sol

196:    if (excessPayment > 0)

221:    if (budgetLeft > 0) 

381:    if (listedId > 0) 

```
