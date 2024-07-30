The protocol in calculating its revenue either through `devFee` as set in `EntityForging` contract here https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L146 and `nukeFundContributions` in `EntityTrading` as https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L72, it rounds down in the calculation. 

`nukeFundContribution` is calculated as `uint256 nukeFundContribution = msg.value / taxCut;` and `devFee` is calculated as `uint256 devFee = forgingFee / taxCut;`. The current `taxCut` is set to `10`. The operation rounds down which ultimately means the amount in `nukeFundContribution` and `devFee` that is to be paid can be manipulated by the seller to ensure it disadvantages the protocol. 

This is particularly an issue in `buyNFT` where a user can set a low amount on NFT.

The project should consider rounding up the two calculation to always favor the protocol