# Non Critical Issues

# 1: Spelling Errors

Vulnerability details

## Proof of Concept

> ***File: EntityTrading.sol***

lsit can be changed to list in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L37

acivte can be changed to active in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L97

canel can be changed to cancel in the following string.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L100

> ***File: EntropyGenerator.sol***

initalize and transcations can be changed to  initialize and transactions in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L46

puroposed can be changed to purposes in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L130

baed and entrtopy can be changed to based and entropy in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L135

calcualtions and entropyto can be changed to calculations and entropy to in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L151

exmaple can be changed to example in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L156

caculated can be changed to calculated in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L184

te calcualte can be changed to to calculate in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L187

he can be changed to the in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L197


> ***File: TraitForgeNft.sol***

distrubted can be changed to distributed in the following comment.

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L357


### Tools Used

Manual Analysis


# 2: FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGE

Vulnerability details

## Context:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces to allow us to apply this rule better.

## Proof of Concept

### Contracts

> ***File: All files in scope***

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L4-L7

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L4-L8

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L4-L9

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L4-L6

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L4-L9

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L4-L12 

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

import {contract1 , contract2} from "filename.sol";

#### Example:

import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";


# 3: ADD TO BLACKLIST FUNCTION

Vulnerability details

## Context:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.


Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code
	
## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Add to Blacklist function and modifier.

> ***Example***

    modifier nonBlacklistRequired(address extension) {
        require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
        _;
    }


# 4: PROJECT UPGRADE AND STOP SCENARIO SHOULD BE

Vulnerability details

## Context:

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.


For reference, see 
https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Create upgrade and stop scenario


# 5: FLOATING PRAGMA SHOULD BE AVOIDED

Vulnerability details

## Context:

FLOATING PRAGMA SHOULD BE AVOIDED

## Proof of Concept


### 1 Result - 6 Instances

> ***File: All files in scope***

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L2

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Lock pragma versions

# Low Impact Vulnerabilities

# 1: Solidity version 0.8.23 won't work on all chains due to MCOPY

Vulnerability details

## Context:

Solidity version 0.8.23 introduces the MCOPY opcode, this may not be implemented on all chains and L2 thus reducing the portability and compatibility of the code. Consider using an earlier solidity version.


## Proof of Concept


### 1 Result - 6 Instances

> ***File: All files in scope***

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/DevFund/DevFund.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityTrading/EntityTrading.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/NukeFund/NukeFund.sol#L2

https://github.com/jauvany/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L2

## Tools Used

Manual Analysis

