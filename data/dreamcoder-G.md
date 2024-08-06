## code 
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L87

function safeRewardTransfer(
    address to,
    uint256 amount
  ) internal returns (uint256) {
    uint256 _rewardBalance = payable(address(this)).balance;
    ...
  }

## Issue
address(this).balance
This expression accesses the balance of the current contract.
It is used to get the amount of Ether (in wei) that the contract currently holds.
It does not require the address to be payable.
Can use like this.
uint256 _rewardBalance = address(this).balance;
