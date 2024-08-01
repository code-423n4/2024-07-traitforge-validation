```oneYear``` variable in ```EntityForging.sol``` does not truly represent what it intends to. As we read the varible name one would think it refers to the lapse time in seconds represented by a year, in which case it should be a constant variable.

What this variable indicates is the required time to set the forgingCounts[] mapping value of a tokenId to 0, as the ```_resetForgingCountIfNeeded``` shows. Therefore the name of the variable should be changed to somewhat more representative, as well as the ```oneYear``` memory variable in the function:

```solidity
uint256 public oneYearInDays = 365 days;
.
.
.
function _resetForgingCountIfNeeded(uint256 tokenId) private {
    uint256 oneYear = oneYearInDays;
    if (lastForgeResetTimestamp[tokenId] == 0) {
      lastForgeResetTimestamp[tokenId] = block.timestamp;
    } else if (block.timestamp >= lastForgeResetTimestamp[tokenId] + oneYear) {
      forgingCounts[tokenId] = 0; // Reset to the forge potential
      lastForgeResetTimestamp[tokenId] = block.timestamp;
    }
  }
```