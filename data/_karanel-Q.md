
# [L-1] `EntityForging::fetchListings` function returns an unbounded array can become very slow due to empty listings

When listings are created `listingCount` increases. When forging is done, this listing is cancelled. This listing can also be cancelled manually and would delete the data at the particular listing index. Since there could be 100k total NFTs out of which possibly 33333 are Forgers (actual count may differ due to forging), and these forgers can be listed multiple times the value of `listingCount` variable could very easily reach the range of 100k-200k.
Even a malicious actor could repeatedly list and cancel the listing to pump the length of this array.

Now to return a list of150k `Listing` struct, where each is 128 bytes, how much memory would be returned?

Length  : Memory
8       : 1KB
8192    : 1MB
150k    : 18.31MB

Retrieving this would be very slow, a better approach would be to implement pagination logic.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L48
```javascript
  function fetchListings() external view returns (Listing[] memory _listings) {
    _listings = new Listing[](listingCount + 1);
    for (uint256 i = 1; i <= listingCount; ++i) {
      _listings[i] = listings[i];
    }
  }
```


# [L-2] Incorrect check for `currentSlotIndex` in `EntropyGenerator::getNextEntropy` and `EntropyGenerator::getEntropy` functions

[Here](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L101) and [here](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L164), `currentSlotIndex` should be from 0 to 769 (total 770 slots). The condition `currentSlotIndex <= maxSlotIndex` should be changed to `currentSlotIndex < maxSlotIndex`

```javascript
  function getNextEntropy() public onlyAllowedCaller returns (uint256) {
    require(currentSlotIndex <= maxSlotIndex, 'Max slot index reached.');
    uint256 entropy = getEntropy(currentSlotIndex, currentNumberIndex);
```
```javascript
  function getEntropy(
    uint256 slotIndex,
    uint256 numberIndex
  ) private view returns (uint256) {
    require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');

```

# [L-3] Missing `DevShareDistributed` event in `NukeFund::receive` function

`DevShareDistributed` event is emitted when the funds are transferred to `DevFund` or `DAOFund` contracts, but not emitted when it is transferred to the owner.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L40-L61
```javascript
  receive() external payable {
    uint256 devShare = msg.value / taxCut; // Calculate developer's share (10%)
    uint256 remainingFund = msg.value - devShare; // Calculate remaining funds to add to the fund

    fund += remainingFund; // Update the fund balance

    if (!airdropContract.airdropStarted()) {
      (bool success, ) = devAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    } else if (!airdropContract.daoFundAllowed()) {
      (bool success, ) = payable(owner()).call{ value: devShare }('');
      require(success, 'ETH send failed');
@>    // @audit-low missing event
    } else {
      (bool success, ) = daoAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    }

    emit FundReceived(msg.sender, msg.value); // Log the received funds
    emit FundBalanceUpdated(fund); // Update the fund balance
  }
```
