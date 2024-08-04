https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/EntropyGenerator/EntropyGenerator.sol#L152


Incorrect and/or different calculation for nuke factor. 

The documentation and "NukeFund::calculateNukeFactor" calculate the nuke factor by `entropy / 40`. However, `EntropyGenerator::deriveTokenParameters` calculates the nuke factor by `entropy / 4000000` - always resulting in the nuke factor being 0. 