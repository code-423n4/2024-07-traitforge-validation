## Title:
1. Incorrect Boundary Check in Entropy Generation Logic

## Description:
The boundary check for currentSlotIndex in the getNextEntropy function incorrectly allows currentSlotIndex to be equal to maxSlotIndex, potentially leading to out-of-bounds errors. The check should ensure currentSlotIndex is strictly less than maxSlotIndex to prevent accessing an invalid index.
```
    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
```

## Impact:
The incorrect boundary check can lead to attempts to access an invalid index in the entropySlots array, potentially causing out-of-bounds errors. This can result in runtime exceptions, contract malfunctions, and potential security vulnerabilities if not properly handled.

## Recommendation:
Change the condition to ensure currentSlotIndex is strictly less than maxSlotIndex:
```
require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
```

## Title
2. Lack of Bid Functionality for Merger NFTs

## Vulnerability details
The current implementation allows forger NFTs to list themselves for forging and merger NFTs to initiate forging by paying the listed fee. 

However, there is no functionality for merger NFT owners to proactively make an offer to forger NFTs. 

This restricts the interaction model and limits potential opportunities for negotiation and better market dynamics.

## Impact
Limited Market Interaction: 
The absence of a bid functionality means that merger NFT owners cannot directly propose terms to forger NFT owners, reducing potential interactions and negotiations in the marketplace.

Reduced User Engagement: 
Users may find the system less engaging due to the lack of flexibility in how they can interact with each other and negotiate terms for forging.

Opportunity Cost: 
Potential transactions may be missed because merger NFT owners cannot signal their interest through bids, leading to lower liquidity and transaction volume in the marketplace.

##  Recommended Mitigation Steps:
Add a feature that allows merger NFT owners to make offers to forger NFT owners and forger NFT owners to accept or reject offers from merger NFT owners.
