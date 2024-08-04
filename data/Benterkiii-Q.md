| Risk   | Issues Details                                                                      | Number        |
|--------|-------------------------------------------------------------------------------------|---------------|
| [L-**] | if the ´ageMultiplier´ isn't set yet it leads to unexpected behaviors               |               |
| [L-**] | if the ´nukeFundAddress´ isn't set yet it leads to a value loss                     |               |

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

## [L-**] if the ´nukeFundAddress´ isn't set yet it leads to a value loss 

#### Description
if the ´nukeFundAddress´ isn't set:
in `EntityForging::forgeWithListed()` the `devFee` will be sended to the 0 address,
in `TraitForgeNft::_distributeFunds()` the `mintPrice` passed in will be sended to the 0 address.

#### Poc
> Ref: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L156
> Ref: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L361
> Ref: https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L308

#### Recommended Mitigation Steps
in `forgeWithListed()` and `_distributeFunds()` add a require statement that make sure the ´nukeFundAddress´ != address(0).