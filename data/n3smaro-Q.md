In the `EntropyGenerator` contract, entropy values are initialized using three separate functions:
- `writeEntropyBatch1` for the first batch.
- `writeEntropyBatch2` for the second batch.
- `writeEntropyBatch3` for the final batch.

Each function handles a specific range of entropy slots and ensures the values are unique and valid. To optimize and reduce contract code size, these functions can be consolidated into a single function that takes a `batchNumber` parameter.

**Proposed Optimization:**

Create a `writeEntropyBatch` function that accepts a `batchNumber` parameter. This function will handle the logic for all batches, determining the appropriate range of entropy slots to initialize based on the provided batch number. By consolidating the batch initialization logic into a single function, this optimization reduces code redundancy, lowers contract size, and simplifies entropy value management.
