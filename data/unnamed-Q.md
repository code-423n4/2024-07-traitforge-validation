
## L-01 :: Lack of Input Validation in `DevFund::updateDev` Function Causes Inefficient Updates

### Summary
The `updateDev` function in the FundDev smart contract does not validate if the new weight differs from the current weight, leading to unnecessary updates.

### Vulnerability Details
#### Lack of Input Validation in `updateDev` Function
- **Explanation:** The `updateDev` function does not check if the new weight is different from the current weight, which may result in unnecessary updates and potential misuse.
- **Affected Function:**
    ```solidity
    function updateDev(address user, uint256 weight) external onlyOwner {
        DevInfo storage info = devInfo[user];
        require(weight > 0, 'Invalid weight');
        require(info.weight > 0, 'Not dev address');
        totalDevWeight = totalDevWeight - info.weight + weight;
        info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;
        info.rewardDebt = totalRewardDebt;
        info.weight = weight;
        emit UpdateDev(user, weight);
    }
    ```

### Impact
There is no effect if the function is called with the same weight, leading to a waste of gas and transaction fees.


### Tools Used
- Manual code review

### Recommendations
Add a check to ensure the new weight is different from the current weight before proceeding with the update. This will prevent unnecessary updates and improve the efficiency of the function.

```solidity
function updateDev(address user, uint256 weight) external onlyOwner {
    DevInfo storage info = devInfo[user];
    require(weight > 0, 'Invalid weight');
    require(info.weight > 0, 'Not dev address');
 ++ require(weight != info.weight, 'Weight is the same as the current value');
    totalDevWeight = totalDevWeight - info.weight + weight;
    info.pendingRewards += (totalRewardDebt - info.rewardDebt) * info.weight;
    info.rewardDebt = totalRewardDebt;
    info.weight = weight;
    emit UpdateDev(user, weight);
}
```

## L-02 Lack of Input Validation in `EntityForging::setTaxCut` Function

### Summary
The `setTaxCut` function does not validate if `_taxCut` is within the range of 0 to 100, potentially leading to a denial of service  if the value exceeds 100.

### Vulnerability Details
In the `setTaxCut` function, there is no check to ensure that the `_taxCut` parameter is between 0 and 100. If an owner sets `_taxCut` to a value greater than 100, it can cause issues with fee calculations, potentially leading to a DoS 

### Impact
If `_taxCut` is set to a value greater than 100, it will lead to incorrect fee calculations, causing transactions to fail and resulting in a denial-of-service (DoS) for users attempting to interact with the contract.

### Tools Used
Manual code review.

### Recommendations
Add input validation to ensure that `_taxCut` is within the range of 0 to 100.

```solidity
// @audit _taxCut should be between 0 and 100
function setTaxCut(uint256 _taxCut) external onlyOwner {
    require(_taxCut >= 0 && _taxCut <= 100, "Invalid tax cut percentage");
    taxCut = _taxCut;
}
```

## L-03 Potential Manipulation of `block.timestamp` Across Contract Functions

### Summary
The contract relies on `block.timestamp` for various time-based logic, making it susceptible to minor manipulations by miners.

### Vulnerability Details
Multiple functions in the contract use `block.timestamp` for critical time-based operations, including resetting counts, minting logic, and other timestamp-based conditions. Since `block.timestamp` can be influenced by miners within a small range, this reliance introduces a potential vulnerability where miners could manipulate the timing of these operations.

### Impact
Miners can slightly manipulate the `block.timestamp` to influence the timing of operations dependent on it. This could result in unintended behavior, such as premature or delayed resets, incorrect calculations, or exploited timing-based conditions.

### Tools Used
Manual code review.

### Recommendations
use Oracle .

## L-04 Handling Edge Case for Zero in `EntropyGenerator::numberOfDigits` Function

### Summary
The `numberOfDigits` function calculates the number of digits in a given number but does not handle the edge case where the input number is zero.

### Vulnerability Details
The `numberOfDigits` function does not account for the scenario where the input number is zero. In this case, the loop will not execute, and the function will return zero digits, which is incorrect. The correct number of digits for zero should be one.

### Impact
If the input number is zero, the function will return 0 instead of 1. This could lead to incorrect results when the number of digits is used for further calculations or conditions.

### Tools Used
Manual code review.

### Recommendations
Add a check for the zero value at the beginning of the function to ensure that it returns 1 for zero. Here is the updated function:

```solidity
function numberOfDigits(uint256 number) private pure returns (uint256) {
    // Handle edge case where the number is zero
 ++ if (number == 0) {
 ++     return 1;
 ++ }

    uint256 digits = 0;
    while (number != 0) {
        number /= 10;
        digits++;
    }
    return digits;
}
```

This update ensures that the function correctly returns 1 when the input number is zero, aligning with the expected behavior of digit counting.


## L-05 Incomplete Address Validation and State Updates in Fallback Function

### Summary

The fallback function for receiving ETH does not verify if the `devAddress` and `daoAddress` are set before attempting to transfer funds. Additionally, it fails to update the state with information about the sender of the ETH.

### Vulnerability Details

- **Issue:** The fallback function does not check whether `devAddress` and `daoAddress` have been set before attempting to transfer ETH. This could result in failed transactions if these addresses are not properly initialized. Furthermore, the function does not update any state related to the `msg.sender`.

- **Affected Function:** `receive()`
  ```solidity
  receive() external payable {
    uint256 devShare = msg.value / taxCut; // Calculate developer's share
    uint256 remainingFund = msg.value - devShare; // Calculate remaining funds to add to the fund

    fund += remainingFund; // Update the fund balance

    if (!airdropContract.airdropStarted()) {
      (bool success, ) = devAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    } else if (!airdropContract.daoFundAllowed()) {
      (bool success, ) = payable(owner()).call{ value: devShare }('');
      require(success, 'ETH send failed');
    } else {
      (bool success, ) = daoAddress.call{ value: devShare }('');
      require(success, 'ETH send failed');
      emit DevShareDistributed(devShare);
    }

    emit FundReceived(msg.sender, msg.value); // Log the received funds
    emit FundBalanceUpdated(fund); // Update the fund balance
  }
  ```

- **Missing Checks:** The function does not check if `devAddress` or `daoAddress` is set. This could lead to failed transactions if these addresses are not properly initialized. Additionally, it does not update any state related to the `msg.sender`.

### Impact

The impact includes the potential for ETH transfers to fail if `devAddress` or `daoAddress` is not properly set.

### Tools Used

- Manual Code Review

### Recommendations

 **Address Validation:** Add checks to ensure `devAddress` and `daoAddress` are set before attempting to transfer ETH. Consider adding a require statement to validate that these addresses are not the zero address.

