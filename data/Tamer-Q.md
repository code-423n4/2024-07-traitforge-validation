## [L-01] There is a chance that multiple golden gods can be claimed within 1 generation

Writing the entropy batches generate pseudo random numbers with 78 digits each.
Each entropy has 6 digits inside this 78 digit number or with other words there are 13 entropies within the big number.
There is a very low chance that this pseudo random number generates 999.999 number from 0 to 6 digits or from 6 to 12 digits or anywhere within the 78 digit number. The 999.999 has to be placed within step of 6, since 1 entropy number has 6 digits.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L181

If the `numberIndex` is 1, then our position will be `1 * 6 = 6`

For example we have this number as a slot value:
`123456999999789012345678901234567890123456789012345678901234567890123456789012`

This math will return the 999.999 number as entropy `uint256 entropy = (slotValue / (10 ** (72 - position))) % 1000000;`
The padded entropy will multiply the current entropy by `1` ,since `10 ** 0 = 1`

## Recommendations:
You could add another state variable inside `EntropyGenerator.sol` and on each generated pseudo number you should make sure that out of all 3 functions which writes the etropy batches there is only 1 number with 999.999