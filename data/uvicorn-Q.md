No zero check in actions `contracts/NukeFund/NukeFund.sol`
```sol
  function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
  }
  receive() external payable {
    uint256 devShare = msg.value / taxCut; // revert
    uint256 remainingFund = msg.value - devShare;
    // ...
  }
```
could cause `receive` failure every time