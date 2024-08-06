# [GAS-1] Consider using `assembly` for ETH transfers

Saves `100-200` gas per transfer.

Example of how it can be done:

```solidity
// Before
function safeRewardTransfer(address to, uint256 amount) internal returns (uint256) {
    uint256 _rewardBalance = payable(address(this)).balance;
    if (amount > _rewardBalance) amount = _rewardBalance;
    (bool success,) = payable(to).call{value: amount}("");
    require(success, "Failed to send Reward");
    return amount;
}

// After
function safeRewardTransfer(address to, uint256 amount) internal returns (uint256) {
    uint256 _rewardBalance = address(this).balance;
    if (amount > _rewardBalance) amount = _rewardBalance;
    
    assembly {
        // `call` returns 0 on error (i.e. out of gas) and 1 on success
        let success := call(gas(), to, amount, 0, 0, 0, 0)
        if iszero(success) {
            // Revert with "Failed to send Reward"
            mstore(0x00, 0x08c379a000000000000000000000000000000000000000000000000000000000)
            mstore(0x04, 0x0000000000000000000000000000000000000000000000000000000000000020)
            mstore(0x24, 0x0000000000000000000000000000000000000000000000000000000000000016)
            mstore(0x44, 0x4661696c656420746f2073656e6420526577617264000000000000000000000000)
            revert(0, 0x84)
        }
    }
    
    return amount;
}
```

<details>

<summary> Instances(11) </summary>

```bash
Found in DevFund/DevFund.sol
20:  (bool success, ) = payable(owner()).call{ value: remaining }('');
        require(success, 'Failed to send Ether to owner');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L20)

```bash
Found in DevFund/DevFund.sol
24:  (bool success, ) = payable(owner()).call{ value: msg.value }('');
      require(success, 'Failed to send Ether to owner');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L24)

```bash
Found in DevFund/DevFund.sol
83: (bool success, ) = payable(to).call{ value: amount }('');
    require(success, 'Failed to send Reward');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L83)

```bash
Found in EntityForging/EntityForging.so
156:    (bool success, ) = nukeFundAddress.call{ value: devFee }('');
        require(success, 'Failed to send to NukeFund');
        (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');
        require(success_forge, 'Failed to send to Forge Owner');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L156-L159)

```bash
Found in EntityTrading/EntityTrading.sol
77:    (bool success, ) = payable(listing.seller).call{ value: sellerProceeds }(
      ''
    );
    require(success, 'Failed to send to seller');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L77C1-L80C50)

```bash
Found in EntityTrading/EntityTrading.sol
114: (bool success, ) = nukeFundAddress.call{ value: amount }('');
     require(success, 'Failed to send Ether to NukeFund');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L114C5-L115C58)

```bash
Found in NukeFund/NukeFund.sol
(bool success, ) = devAddress.call{ value: devShare }('');
require(success, 'ETH send failed');
(bool success, ) = payable(owner()).call{ value: devShare }('');
require(success, 'ETH send failed');
(bool success, ) = daoAddress.call{ value: devShare }('');
require(success, 'ETH send failed');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L47-L55)

```bash
Found in NukeFund/NukeFund.sol
(bool success, ) = payable(msg.sender).call{ value: claimAmount }('');
require(success, 'Failed to send Ether');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L177C5-L178C46)

```bash
Found in TraitForgeNft/TraitForgeNft.sol
(bool refundSuccess, ) = msg.sender.call{ value: excessPayment }('');
require(refundSuccess, 'Refund of excess payment failed.');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L197C7-L198C66)

```bash
Found in TraitForgeNft/TraitForgeNft.sol
(bool refundSuccess, ) = msg.sender.call{ value: budgetLeft }('');
require(refundSuccess, 'Refund failed.');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L222C7-L223C48)

```bash
Found in TraitForgeNft/TraitForgeNft.sol
(bool success, ) = nukeFundAddress.call{ value: totalAmount }('');
require(success, 'ETH send failed');
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L361C5-L362C41)

</details>

# [GAS-2] Optimize Storage Usage by using `uint128` or smaller for storage variables that are not expected to have high values

The storage variables in `EntropyGenerator.sol` and `NukeFund.sol` can be optimized to use types lower than `uint256`, so that they can be combined in single storage slots.

`20,000+` gas saver per storage write to a previously unused slot.

<details>

<summary> Instances(2) </summary>

```solidity
 uint256[770] private entropySlots; // Array to store entropy values
  uint256 private lastInitializedIndex = 0; // Indexes to keep track of the initialization and usage of entropy values
  uint256 private currentSlotIndex = 0;
  uint256 private currentNumberIndex = 0;
  uint256 private batchSize1 = 256;
  uint256 private batchSize2 = 512;
  // Constants to define the limits for slots and numbers within those slots
  uint256 private maxSlotIndex = 770;
  uint256 private maxNumberIndex = 13;
  uint256 public slotIndexSelectionPoint;
  uint256 public numberIndexSelectionPoint;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L10C2-L20C44)

```solidity
uint256 public constant MAX_DENOMINATOR = 100000;

  uint256 private fund;
  ITraitForgeNft public nftContract;
  IAirdrop public airdropContract;
  address payable public devAddress;
  address payable public daoAddress;
  uint256 public taxCut = 10;
  uint256 public defaultNukeFactorIncrease = 250;
  uint256 public maxAllowedClaimDivisor = 2;
  uint256 public nukeFactorMaxParam = MAX_DENOMINATOR / 2;
  uint256 public minimumDaysHeld = 3 days;
  uint256 public ageMultiplier;
```

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/NukeFund/NukeFund.sol#L12-L24)

</details>

# [GAS-3] Functions that are only called by the owner, could be used without `onlyOwner` modifier, with a simple `require(msg.sender == owner())`

Replace the `onlyOwner` modifier with a direct require check in functions that are only called by the owner.

`20-40` gas saved per call.

# [GAS-4] Use Assembly for Efficient Arithmetic in `EntropyGenerator.sol`

Use assembly for complex arithmetic operations.

```solidity
// Before
uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000;

// After
assembly {
    let shifted := div(slotValue, exp(10, sub(72, position)))
    entropy := mod(shifted, 1000000)
}
```

Variable, potentially `50-200` gas saved per operation.

# [GAS-5] Use Short-Circuiting in `EntityForging.sol`

Order conditions in if statements from least to most gas-consuming.

```solidity
// Before
require(
    nftContract.getApproved(tokenId) == address(this) || nftContract.isApprovedForAll(msg.sender, address(this)),
    "Contract must be approved to transfer the NFT."
);

// After
require(
    nftContract.isApprovedForAll(msg.sender, address(this)) || nftContract.getApproved(tokenId) == address(this),
    "Contract must be approved to transfer the NFT."
);
```