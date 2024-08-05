### DevFund::updateDev()
`updateDev()` function is not available via `IDevFund` interface. If the system uses `IDevFund` interface, it will not have access to `updateDev()`.  But, it can be accessed via `DevFund` contract instance.

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

### NukeFund::setTaxCut() should not allow 0 to be set for `taxCut`
While this is owner controller,the risk is very low, it is recommended to validate the taxCut to greater than 0 to prevent DOS when funds are transferred directly to the contract.

```
   function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
  }
```

The receive function will revert due to the below logic.

```
   receive() external payable {
    uint256 devShare = msg.value / taxCut; //@audit, taxCut should not be 0
    ...
   }
```

### EntityTrading::setTaxCut() should not allow 0 to be set for `taxCut`
While this is owner controller,the risk is very low, it is recommended to validate the taxCut to greater than 0 to prevent DOS when calling `buyNFT` function.

```
  function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
  }

  function buyNFT(uint256 tokenId) external payable whenNotPaused nonReentrant {
    ....
    uint256 nukeFundContribution = msg.value / taxCut;
    ... 
```

### EntityTrading::listNFTForSale() / EntityTrading::buyNFT()
`EntityTrading` contract will not function properly for NFTs listed in the market by a smart contract that does not receive Ether. Ar the time of listing the NFT, there will be no problem. But, when the user attempts to buy the NFT, the `buyNFT()` function tries to send Ether to the seller contract. 

If seller contract is a smart contract that does not accept ether, it will revert.
