### LOW-1: Make OneYearInDays a Constant to Prevent Inconsistent Behavior 
### Description
In the EntityForging there is a variable "oneYearInDays". A setter is also defined to set this variable. 
Since the days in an year is a constant value , there is no need to define a setter function as changing the oneYearInDays would be a phishy thing. It would affect the _resetForgingCountIfNeeded() since the following condition will behave differently of the values on oneYear.
```js
else if (block.timestamp >= lastForgeResetTimestamp[tokenId] + oneYear) 
```
Reference to Code
1. https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L14
2. https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L40-L42
3. https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L199-L208

### Recommendation
It is recommended to make the value constant & function to be removed


### LOW-2: fetchListing() function might lead to failure 
### Description:
In the EntityForging contract, when a forger lists a token for forging by calling listForForging(), the listingCount is incremented. However, when a listing is cancelled, the listingCount is not decremented. Instead, the values at listings[listedTokenIds[tokenId]] are deleted (set to default values), but the overall size of the listings mapping remains the same.

This causes an issue when calling the fetchListings() function, which fetches listings based on listingCount. If there are a large number of listings and cancellations, the listingCount will not accurately reflect the active listings, leading to potential failures or inefficiencies in fetchListings().
#### POC
1. A user lists a token by calling listForForging(), increasing listingCount.
2. The user then cancels the listing. The listingCount remains unchanged, but the corresponding entry in listings is set to default values.
3. Over time, the listings mapping grows larger as tokens are repeatedly listed and unlisted, causing fetchListings() to become inefficient or fail due to iterating over a large number of invalid entries.

Reference to Code:
1. https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L48-L53
2. https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L177-L197

### Recommendation
To address this issue, consider implementing a mechanism to properly track active listings and adjust the listingCount accordingly.