## There is mismatch in calculation of `forgePotential` in code at different places. 
In `EntityForging`, the `forgePotential` is calculated as following:
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L86
```solidity
uint8 forgePotential = uint8((entropy / 10) % 10); // Extract the 5th digit from the entropy
```
It says that `forgePotential` is 5th digit of the entropy.

In function `deriveTokenParameters` of `EntropyGenerator` contract, the `forgePotential` is calculated as follows:
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L153
```solidity
forgePotential = getFirstDigit(entropy);
```
It says that `forgePotential` is 1st digit of the entropy.



## Extra fee not refunded during forging

The `merger` can send more ether than `forgingFee` when calling `forgeWithListed`. Extra ether is not refunded to him but it is distributed to `nukeFund` and `forger`. This will cause loss to `merger` if he sends fee more than `forgingFee`.

https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityForging/EntityForging.sol#L126
```solidity
require(msg.value >= forgingFee, 'Insufficient fee for forging');
```
