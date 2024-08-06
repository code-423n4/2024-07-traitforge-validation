## Title 
Combine Mappings for `Listing` Information in `EntityForging` Contract

## Description

In the `EntityForging` contract, two separate mappings are used to manage listing information:

- `listedTokenIds` maps token IDs to listing indices.
- `listings` maps indices to Listing structs, which contain details such as address, token ID, and fee.

To optimize gas usage, these two mappings can be combined into a single mapping. By mapping directly from the token ID to the Listing struct, both the token ID and listing details can be accessed in one step, reducing storage overhead and improving gas efficiency.