## Consolidate Multiple Token Metadata Mappings into a Single Mapping with Struct
In the `TraitForgeNft` contract, multiple mappings are used to manage token metadata:

- `tokenCreationTimestamps` for creation timestamps.
- `lastTokenTransferredTimestamp` for the last transfer timestamp.
- `tokenEntropy` for entropy values.
- `generationMintCounts` for mint counts by generation.
- `tokenGenerations` for token generations.
- `initialOwners` for initial owner addresses.

To optimize gas usage and simplify data management, these mappings can be consolidated into a single mapping using a struct that groups all related metadata. This approach reduces the number of storage operations and improves contract efficiency.

**Proposed Optimization:**

1. **Define a Struct:** Create a `TokenMetadata` struct to hold all token-related data:
    ```solidity
    struct TokenMetadata {
        uint256 creationTimestamp;
        uint256 lastTransferredTimestamp;
        uint256 entropy;
        uint256 generationMintCount;
        uint256 generation;
        address initialOwner;
    }
    ```

2. **Replace Existing Mappings:** Use a single mapping to manage token metadata:
    ```solidity
    mapping(uint256 => TokenMetadata) public tokenMetadata;
    ```

By consolidating all metadata into one mapping, this optimization reduces gas costs associated with multiple separate mappings and simplifies the contract's data structure.

## Combine Mappings for `Listing` Information in `EntityForging` Contract
In the `EntityForging` contract, two separate mappings are used to manage listing information:

- `listedTokenIds` maps token IDs to listing indices.
- `listings` maps indices to Listing structs, which contain details such as address, token ID, and fee.

To optimize gas usage, these two mappings can be combined into a single mapping. By mapping directly from the token ID to the Listing struct, both the token ID and listing details can be accessed in one step, reducing storage overhead and improving gas efficiency.
