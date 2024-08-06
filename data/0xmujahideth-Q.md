## Title
ERC-721 transfers do not use ``safeTransferFrom ``
 ## Lines of code
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L81
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L104
## Vulnerability details
## Impact
The cases where ERC-721 tokens are transferred are always using transferFrom and not safeTransferFrom. This means users need to be aware their tokens may get locked up in an unprepared contract recipient.

This may be intentional to reduce the potential for reentrancy, but it may strike users unprepared.

If it is by design, then it means users must be educated about this fact and the frontend would need to verify target addresses prior to submitting any transactions and hope that other frontends/integrations do not exists.

Should a ERC-721 compatible token be transferred to an unprepared contract, it would end up being locked up there. 
## Tool Used
Manual Review
## Recommended Mitigation
Consider using ``safeTransferFrom``.