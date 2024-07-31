1. Gas Optimization in mintWithBudget() Function
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L212
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/TraitForgeNft/TraitForgeNft.sol#L217
In the mintWithBudget function, there is a local variable amountMinted that is incremented within a loop but never utilized after the loop. Removing this unused variable can result in minor gas savings, as each read and write operation to the variable costs gas. Even small optimizations can add up, particularly in high-frequency operations.
2. Incorrect Use of `storage` Instead of `memory` in cancelListing() Function
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntityTrading/EntityTrading.sol#L95