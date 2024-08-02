# Function `EntropyGenerator::deriveTokenParameters` implements incorrect logic for users' parameters calculation

## Impact
This function is not utilized in any of the provided files, indicating its likely use for off-chain monitoring activities or frontend dynamics. Therefore, it does not pose a material risk to the protocol, and is categorized as a low-severity issue.

In this function, the initial `nukeFactor` and the `forgePotential` are calculated incorrectly:

```javascript
nukeFactor = entropy / 4000000; 
forgePotential = getFirstDigit(entropy); 
```  

The `nukeFactor` will always be 0 since the entropies are 6 digits long, and `forgePotential` is calculated as the 5th digit in the rest of the functions, whereas here it is incorrectly taken as the first digit.

## Recommended Mitigation
Align the calculations with the rest of the functions to ensure accurate values are displayed.