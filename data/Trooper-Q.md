## [L-01] EntropyGenerator: No check for entropy 999,999 in batch3

- File:     contracts/EntropyGenerator/EntropyGenerator.sol
- Contract: EntropyGenerator.sol           
- Function: writeEntropyBatch3
- Line:     84-98
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L84-L98

`slotIndexSelectionPoint` and `numberIndexSelectionPoint` set a fixed value for the best entropy 999,999 [see](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L170-L175). In `writeEntropyBatch3` there is no check for another 999,999 entry. Even though its unlikely, its possible to have more than one golden god.

## [L-02] EntropyGenerator: Wrong value for forge potential

- File:     contracts/EntropyGenerator/EntropyGenerator.sol
- Contract: EntropyGenerator.sol           
- Function: deriveTokenParameters
- Line:     153
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L153

Wrong value for forgePotential. Per whitepaper the forge potential is the 5th number of the entropy. That is also how it is implemented in [EntityForging](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L86).

## [L-03] EntropyGenerator: Wrong input validation

- File:     contracts/EntropyGenerator/EntropyGenerator.sol
- Contract: EntropyGenerator.sol           
- Function: getEntropy
- Line:     168
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L168

The function allows fetching a `slotIndex` lower or equal to `maxSlotIndex` (770). If called with 770 it does pass the input validation, but reverts with `panic: array out-of-bounds access (0x32)` as entropySlots is length 770 with the highest index at 769 [see](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L90-L94).

## [L-04] EntityForging: User ETH can get stuck

- File:     contracts/EntityForging/EntityForging.sol
- Contract: EntityForging.sol           
- Function: forgeWithListed
- Line:     126
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L126

The function checks if the msg.value is equal or greater as the `forgingFee`. It only uses the `forgingFee` amount and does not return the excess ETH if the user send more as needed. Either convert the check to a more restrictive one, allowing only the exact amount needed, or add a call to return the excess amount at the end.

## [L-05] TraitForgeNft: Missing forwarding functions for Airdrop.sol

- File:     contracts/TraitForgeNft/TraitForgeNft.sol
- Contract: TraitForgeNft.sol           
- Function: -
- Line:     -
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol

TraitForgeNft must be the owner of Airdrop.sol, as it calls the restricted functions [addUserAmount()](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L297) and [subUserAmount(https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L148)](). After transferring the ownership, it is no longer possible to configure the Airdrop contract. The only function forwarded is [startAirdrop()](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L96-L98). 
If the other owner only functions on Airdrop.sol should be callable after transferring the ownership, consider adding a forwarding function on TraitForgeNft , same as for startAirdrop. Alternatively add another access level to the airdrop contract (i.e. onlyOwner & onlyNFT).


## [L-06] TraitForgeNft: Contract might burn ETH

- File:     contracts/TraitForgeNft/TraitForgeNft.sol
- Contract: TraitForgeNft.sol           
- Function: _distributeFunds
- Line:     L361
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L361

There is no check if the `nukeFundAddress` is set. The default value is the zero address. Esp. as whitelisted users can mint immediately after the contract is live (no pause function, calling before the address is set). Consider adding a check if the address is set, otherwise it would burn tokens.


## [L-07] TraitForgeNft: NFTs are transferable 

- File:     contracts/TraitForgeNft/TraitForgeNft.sol
- Contract: TraitForgeNft.sol           
- Function: _beforeTokenTransfer
- Line:     369
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L367-L395

There are no restrictions for transferring the NFTs. As there will be a fee on the TrailForge trading contract, users might use a secondary market to avoid the additional fee (10% of sale price [see](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L72-L73)). Consider limiting the transfer function to protocol only functions if this behavior should not happen.


## [L-08] TraitForgeNft: Entropy after forging is not padded

- File:     contracts/TraitForgeNft/TraitForgeNft.sol
- Contract: TraitForgeNft.sol           
- Function: forge
- Line:     173
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L173

EntropyGenerator makes sure the new entropy is [padded](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L182). When forging two NFTs, the new entropy is averaged between the existing ones. The new entropy is [not padded](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L173). Example: `newEntropy = (100_012 + 100_011) / 2 = 100_023 / 2 = 50_011`.


## [L-09] TraitForgeNft: When nuking/buring an NFT, the original owner loses his aidrop amount

- File:     contracts/TraitForgeNft/TraitForgeNft.sol
- Contract: TraitForgeNft.sol           
- Function: burn
- Line:     148
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L146-L149

The burn function burns the airdrop share of the original owner as mapping is never updated when selling an NFT. Verify if this is intended, as this adds a huge disadvantage at trading NFTs. Also a user might not be aware and feels like he was robbed airdrop shares.


## [L-10] TraitForgeNft: Bad source of randomness allows to predict/abuse the "golden god" entropy of the next generation

- File:     contracts/TraitForgeNft/TraitForgeNft.sol
- Contract: TraitForgeNft.sol           
- Function: _incrementGeneration
- Line:     353
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L345C12-L355

After a generation is minted completely [_incrementGeneration](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L345-L355) is called. This updates the alpha indices. They are updated based on the hash of the last blockhash and the current timestamp. A minter could time his action in a way to generate a golden god in a for him favorable position. Esp. seeing that the "getNextEntopy" index is not updated for a new generation. So he could mint two NFTs in a way that the next index is the golden god of the new generation.


## [L-11] EntityForging: Contract might burn ETH

- File:     contracts/EntityForging/EntityForging.sol
- Contract: EntityForging.sol           
- Function: forgeWithListed
- Line:     156
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L156

There is no check if the `nukeFundAddress` is set. The default value is the zero address. Consider adding a check if the address is set, otherwise it would burn tokens.


## [L-12] NukeFund: Contract will have remaining ETH after the game is done that can never be claimed

- File:     contracts/NukeFund/NukeFund.sol
- Contract: NukeFund.sol           
- Function: -
- Line:     -
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol

Nuking a NFT pays [at most 50% of the current funds](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L167) in the NukeFund. This means that it will keep some ETH forever. Consider adding a cleanup function that sends the remaining ETH to e.g. the DAO.


## [L-13] NukeFund: Different definitions of maturity

- File:     contracts/NukeFund/NukeFund.sol
- Contract: NukeFund.sol           
- Function: canTokenBeNuked
- Line:     191
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L184-L194

The current implementation of token maturity calculations differs across functions, which needs alignment with the whitepaper's definition of aging. Specifically:

- The [whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit) defines aging as: "Every Entity ages by blocktime from their mint date..."
- The [calculateAge](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L122) function computes the maturity of a token based on its creation time.
- The [canTokenBeNuked](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L191) function currently calculates whether a token can be "nuked" by checking if it has reached a minimum maturity of 3 days from the getTokenLastTransferredTimestamp.

To ensure consistency with the whitepaper, the `canTokenBeNuked` function should be updated to base its maturity calculation on the token's mint time instead of the last transferred timestamp. This change aligns with the definition provided, which considers aging from the mint date, ensuring that all maturity calculations are uniform across the system.

## [L-14] NukeFund: Inconsistent Dev Share Distribution

- File:     contracts/NukeFund/NukeFund.sol
- Contract: NukeFund.sol           
- Function: receive
- Line:     51
- Link:     https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L46-L57

Under certain conditions, the development (dev) share is incorrectly sent to the owner of the NukeFund instead of the intended recipient. 

Specifically, the dev share should be send to the dev contract until the airdrop is started. Afterwards the [whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit) states that the dev share should be send to the DAO. However, if the daoFundAllowed setting in the airdrop contract is not set to true, the dev share is erroneously sent to the owner of the NukeFund.

Therefore the current implementation does not reflect these intentions accurately, leading to potential misallocation of the dev share.

