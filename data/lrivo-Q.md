# Loss of fees in `EntityForging:forgeWithListed()` if `msg.value > forgingFee`

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L126

The fees are computed by taking into account the forgingFee rather than the actual msg.value being sent so, any additional ETH, will be forever stuck in the contract.

### Recommended Mitigation
Reject any `msg.value` different from the `forgingFee`:

```diff
function forgeWithListed() {
-   require(msg.value >= forgingFee, 'Insufficient fee for forging');
+   require(msg.value == forgingFee, 'Invalid fee for forging');
}
```