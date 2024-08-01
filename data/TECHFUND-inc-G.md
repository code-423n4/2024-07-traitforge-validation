### EntityForging::forgeWithListed(...)
Any excess ether received into this function should be returned back to the caller after all the process was completed.

```solidity
    require(msg.value >= forgingFee, 'Insufficient fee for forging');
```
Once all the settlements are complete, excess ether should be returned back to the caller.

### EntityForging::cancelListingForForging(...)
When the listing for Forging is cancelled, the logic does not clear the `tokenid` to listings index mapping. As a result, when `getListedTokenIds(...)` is called, it will return an index which is no more valid.

```solidity
mapping(uint256 => uint256) public listedTokenIds;

function getListedTokenIds(
    uint tokenId_
  ) external view override returns (uint) {
    return listedTokenIds[tokenId_];
  }
```