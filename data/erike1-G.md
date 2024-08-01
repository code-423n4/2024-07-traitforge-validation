## Impact

Unnecessary calculations 



 
## Proof of Concept

Target contract : NukeFund.sol



APPROVAL LOGIC ERROR 



There is problem with logic operators and verification used in case of nuke() function

How isApprovedForAll works:

- NFTs (not all your NFTs, but the NFTs you own in the context of this contract).

In the nuke function there 3 require statements 

Proof of concept case:

1st - isApprovedorOwner is executed against the msg.sender
 -msg.sender isApproved to spend the token ,but not own it.That pass the require.

2nd - Considering the || operator isApprovedForAll(msg.sender,address of NukeFund.sol) 
 -executed and return true;

3rd - token can be nuked pass 


The mistake is that when  nftContract.isApprovedForAll(msg.sender, address(this)) is executed at line 165. The msg.sender doesn't own the token.
Resulting in error in the burn function of the of ERC721 for not appropriate permission.Till that all the code is executed and gas is used as follows 


The following is total gas waste report in case of line of code - gas used 


    //Require Statement: ~2,300 gas
    //Calculate Token Age: ~2,300 gas
    //Return Statement: ~200 gas
    // Total : 4,800 gas
    require(canTokenBeNuked(tokenId), 'Token is not mature yet');



    // Gas usage: ~2,300 gas
    uint256 finalNukeFactor = calculateNukeFactor(tokenId); 

    // Gas usage: ~400 gas for each arithmetic operation
    uint256 potentialClaimAmount = (fund * finalNukeFactor) / MAX_DENOMINATOR; 
    uint256 maxAllowedClaimAmount = fund / maxAllowedClaimDivisor; 


    // Gas usage: ~200 gas for the conditional check and assignment
    uint256 claimAmount = finalNukeFactor > nukeFactorMaxParam
      ? maxAllowedClaimAmount
      : potentialClaimAmount;

 
    // Gas usage: ~5,000 gas for storage write
    fund -= claimAmount;

    // 32,700 gas till the revert
    nftContract.burn(tokenId);

Total gas : 45,600

## Tools Used

Manual audit

## Recommended Mitigation Steps

Prevent execution by replacing this line of code (line 163:NukeFund.sol) -  require(
      nftContract.getApproved(tokenId) == address(this) ||
        nftContract.isApprovedForAll(msg.sender, address(this)),
      'Contract must be approved to transfer the NFT.'
    );

With the following 

require(
        nftContract.isApprovedOrOwner((address.this), tokenId),
      'Contract must be approved to transfer the NFT.'
    );
