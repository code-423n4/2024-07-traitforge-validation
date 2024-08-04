| Risk   | Issues Details                                                                      | Number        |
|--------|-------------------------------------------------------------------------------------|---------------|
| [L-**] | if the ´ageMultiplier´ isn't set yet it leads to unexpected behaviors               |               |

## [L-**] if the ´ageMultiplier´ isn't set yet it leads to unexpected behaviors

#### Description
the `calculateAge()` vieew function will always return 0 in case the ´ageMultiplier´ isnt set yet,
leading both the `calculateNukeFactor()` and `nuke()` to revert with unclear reason. 

#### Poc
> Ref: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L131
> Ref: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L143
> Ref: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L165

#### Recommended Mitigation Steps
in `calculateAge()` add a require statement that make sure the ´ageMultiplier´ is > than 0.