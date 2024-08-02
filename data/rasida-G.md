## Gas Optimizations  


## [G-01] Use assembly to check for `address(0)`

```solidity
file: /contracts/common/AddressResolver.sol

81        if (addressManager == address(0)) revert RESOLVER_INVALID_MANAGER();

85        if (!_allowZeroAddress && addr_ == address(0)) {

```