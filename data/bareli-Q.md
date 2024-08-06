 function setTaxCut(uint256 _taxCut) external onlyOwner {
    taxCut = _taxCut;
  }

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L33C2-L35C4


there should be a limit on the taxCut.