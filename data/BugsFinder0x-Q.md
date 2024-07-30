# Impact
It is impossible to create the next-gen of NFT's when the maxTokensPerGen is reached.

This happens because the TraitForgeNFT is not the owner of the EntropyGenerator contract.


## Workflow
1) User1 calls the mintToken() function, and the token minted is the last one for the actual NFT-Gen

2) User2 calls the mintToken() function, and will reach this part of the code from TraitForgeNFT contract:

```
function _mintInternal(address to, uint256 mintPrice) internal {
    if (generationMintCounts[currentGeneration] >= maxTokensPerGen) {
      _incrementGeneration();
    }
    ...
}
```

3) the _incrementGeneration() is called from the TraitForgeNFT Contract

```
function _incrementGeneration() private {
    ...
    entropyGenerator.initializeAlphaIndices();
    ...
  }
```

4) The initializeAlphaIndices() from entropyGenerator contract is called:

```
function initializeAlphaIndices() public whenNotPaused onlyOwner {
    ...
}
```

This function is callable only from the owner of the contract as declared from the modifier.


At this point, User2 gets a revert, because the caller is the TraitForgeNFT address, but the Owner of the entropyGenerator contract is not the TraitForgeNFT address 




## Proof Of Concept
```
it("Mint next Gen Tokens when the top is reached for a previous Gen", async()=>{                                                                      
                                                                                       
    // merkleInfo.whitelist[0] ===> owner.address                                      
    // merkleInfo.whitelist[1] ===> user1.address                                      
    // ...                                                                             
                                                                                       
    // Get Proof from owner                                                            
    proof = merkleInfo.whitelist[0].proof;                                             
                                                                                       
    await nft.mintToken(proof, {value: ethers.utils.parseEther("5")});                 
    console.log(await nft.balanceOf(owner.address));                                   
                                                                                       
    await nft.mintToken(proof, {value: ethers.utils.parseEther("5")});                 
    console.log(await nft.balanceOf(owner.address));                                   
});                                                                                    

```


## Tool Used
Manual


## Mitigation

Transfer the ownership of the entropyGenerator contract to the TraitForgeNFT address.

```
it("Mint next Gen Tokens when the top is reached for a previous Gen", async()=>{                                 
    await entropyGenerator.transferOwnership(nft.address);                                                       
                                                                                                                 
    // merkleInfo.whitelist[0] ===> owner.address                                                                
    // merkleInfo.whitelist[1] ===> user1.address                                                                
    // ...                                                                                                       
                                                                                                                 
    // Get Proof from owner                                                                                      
    proof = merkleInfo.whitelist[1].proof;                                                                       
                                                                                                                 
                                                                                                                 
    await nft.connect(user1).mintToken(proof, {value: ethers.utils.parseEther("5")});                            
    console.log("nftBalance User1", await nft.balanceOf(user1.address));                                         
                                                                                                                 
    await nft.connect(user1).mintToken(proof, {value: ethers.utils.parseEther("5")});                            
    console.log("nftBalance User1", await nft.balanceOf(user1.address));                                         
});                                                                                                              
```