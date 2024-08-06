##Title 
Inefficient `entropySlots` array usage to store the total number of entropy values instead of utilizing a mapping.

## Impact
The contract increases inefficiency in storage and gas consumption by using a fixed-size array (`entropySlots`) to store entropy values. This approach consumes more gas and storage than necessary, especially when accessing or modifying specific values.


## Proof of Concept
Consider the following example:

Using an array:
```solidity
uint256[770] private entropySlots;
```
Accessing or updating an element in this array involves iterating over the array and using more gas for storage operations.

Using a mapping:
```solidity
mapping(uint256 => uint256) private entropySlots;
```
Mappings provide direct access to specific elements, reducing the gas cost associated with storage operations. This is because mappings allow for constant-time complexity (O(1)) for read and write operations, compared to the linear time complexity (O(n)) when dealing with arrays for certain operations.

Gas cost comparison:
- Array access: `SLOAD` operation is more expensive due to the linear search required for some operations.
- Mapping access: `SLOAD` operation is less expensive as it provides direct access.


## Tools Used
Manual review

## Recommended Mitigation Steps
Utilize a mapping from the tokenId to the entropy values to store these values. This will optimize storage efficiency and reduce gas consumption.
