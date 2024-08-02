Issue: Lack of Validation for Zero Address in Developer Management Functions

Severity: Low Risk

Context: DevFund.sol#L30 - addDev Function

Description: The DevFund contract allows the owner to add developers using the addDev function. This function does not explicitly prevent the addition of the zero address (0x0). As a result, the owner could add 0x0 as a developer, which could lead to several issues:

    Unrecoverable Funds: If the contract distributes rewards to the zero address, these funds will be lost permanently since they cannot be retrieved.
    Disrupted Operations: Functions that involve transactions or calculations based on developer addresses may fail or behave unpredictably if 0x0 is used, causing disruptions in normal contract operations.
    Favoring Certain Developers: The owner might use the zero address to simulate a non-existent or neutral developer slot, which could be a tactic to privilege or exclude certain developers. For instance, the owner might manipulate reward distribution or other contract functionalities to benefit specific addresses while rendering others ineffective.[Centralization risks:Direct misuse of privileges shall be submitted in the QA report.]

Recommendation:

    Address Validation: Implement a check in the addDev function to prevent the zero address from being added as a developer.
    

function addDev(address user, uint256 weight) external onlyOwner {
    require(user != address(0), 'Invalid address');  // Prevent zero address
    DevInfo storage info = devInfo[user];
    require(weight > 0, 'Invalid weight');
    require(info.weight == 0, 'Already registered');
    info.rewardDebt = totalRewardDebt;
    info.weight = weight;
    totalDevWeight += weight;
    emit AddDev(user, weight);
}