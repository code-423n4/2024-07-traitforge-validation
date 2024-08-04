#### [L-01] There's no check for exceeding maxGeneration when incrementing generation
 Assuming the generation now is at max generation and when increment generation, current generation will exceed max generation because there's no check for that.

[Link to code](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L345-L350):
```solidity
 function _incrementGeneration() private {
    require(
      generationMintCounts[currentGeneration] >= maxTokensPerGen,
      'Generation limit not yet reached'
    );
    currentGeneration++;
```
