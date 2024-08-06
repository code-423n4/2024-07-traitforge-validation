## L-01
If the ownership of the entropyGenerator contract is not transferred to the nft contract address, minting the new NFT gen is impossible.  


### Impact
It is impossible to create the next-gen of NFT's when the maxTokensPerGen is reached.
This happens because the TraitForgeNFT is not the owner of the EntropyGenerator contract.


### Functions and Contracts Involved
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L206-L216

mintToken()
TraitForgeNft.sol
initializeAlphaIndices()
entropyGenerator.sol



### Workflow
1) User1 calls the mintToken() function, and the token minted is the last one for the actual NFT-Gen.
At this point, will reach this part of the code from TraitForgeNFT contract:


```
function _mintInternal(address to, uint256 mintPrice) internal {
    if (generationMintCounts[currentGeneration] >= maxTokensPerGen) {
      _incrementGeneration();
    }
    ...
}
```

2) the _incrementGeneration() is called from the TraitForgeNFT Contract

```
function _incrementGeneration() private {
    ...
    entropyGenerator.initializeAlphaIndices();
    ...
  }
```

3) The initializeAlphaIndices() from entropyGenerator contract is called:

```
function initializeAlphaIndices() public whenNotPaused onlyOwner {
    ...
}
```

This function is callable only from the owner of the contract as declared from the modifier.


At this point, User2 gets a revert, because the caller is the TraitForgeNFT contract address, but the Owner of the entropyGenerator contract is not the TraitForgeNFT address.




### Proof Of Concept
```
const {expect} = require("chai");
const {expectRevert} = require("@nomicfoundation/hardhat-network-helpers");
const { ethers } = require('hardhat');
const generateMerkleTree = require('../scripts/genMerkleTreeLib');



describe("Impossible mint new NFT-GEN when top is reached", function(){

    let owner, user1, user2, user3, user4, hacker;
    let Trait, trait, TraitNFT, nft, Airdrop, airdrop, EntropyGenerator, entropyGenerator,
        EntityForging, entityForging, DevFund, devFund, NukeFund, nukeFund, DAOFund, daoFund,
        EntityTrading, entityTrading, merkleInfo;

    const ROUTER_ADDRESS = '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D';


    let address, leaf
    let proof = [];


    before(async()=>{

        [owner, user1, user2, user3, user4, hacker] = await ethers.getSigners();

        // Deploy TOKEN ERC20
        Trait = await ethers.getContractFactory("Trait");
        trait = await Trait.deploy("TRAIT", "TRT", 18, ethers.utils.parseEther('1000000'));
        await trait.deployed();


        // Deploy DAOFund
        DAOFund = await ethers.getContractFactory('DAOFund');
        daoFund = await DAOFund.deploy(trait.address, ROUTER_ADDRESS);
        await daoFund.deployed();


        // Deploy NFT
        TraitNFT = await ethers.getContractFactory("TraitForgeNft");
        nft = await TraitNFT.deploy();
        await nft.deployed();


        // Deploy Airdrop
        Airdrop = await ethers.getContractFactory('Airdrop');
        airdrop = await Airdrop.deploy();
        await airdrop.deployed();


        // @Audit da controllare
        await nft.setAirdropContract(airdrop.address);
        await airdrop.transferOwnership(nft.address);


        // Deploy EntropyGenerator
        EntropyGenerator = await ethers.getContractFactory('EntropyGenerator');
        entropyGenerator = await EntropyGenerator.deploy(nft.address);
        await entropyGenerator.deployed();

        await entropyGenerator.writeEntropyBatch1();
        await nft.setEntropyGenerator(entropyGenerator.address);


        // Deploy EntityForging contract
        EntityForging = await ethers.getContractFactory('EntityForging');
        entityForging = await EntityForging.deploy(nft.address);
        await entityForging.deployed();

        await nft.setEntityForgingContract(entityForging.address);


        // Deploy DevFund contract
        DevFund = await ethers.getContractFactory('DevFund');
        devFund = await DevFund.deploy();
        await devFund.deployed();


        // Deploy NukeFund contract
        NukeFund = await ethers.getContractFactory('NukeFund');
        nukeFund = await NukeFund.deploy(
          nft.address,
          airdrop.address,
          devFund.address,
          daoFund.address
        );
        await nukeFund.deployed();
        await nft.setNukeFundContract(nukeFund.address);


        // Deploy EntityTrading contract
        EntityTrading = await ethers.getContractFactory('EntityTrading');
        entityTrading = await EntityTrading.deploy(nft.address);
        await entityTrading.deployed();
        await entityTrading.setNukeFundAddress(nukeFund.address);


        // Generate MarkleTree
        merkleInfo = generateMerkleTree([
          owner.address,
          user1.address,
          user2.address,
          user3.address,
          user4.address,
          hacker.address,

        ]);

        await nft.setRootHash(merkleInfo.rootHash);
    });




    it("Mint next Gen Tokens when the top is reached for a previous Gen supposing that every gen has 2 tokens as max limit", async()=>{        

        // Get Proof from owner
        proof = merkleInfo.whitelist[1].proof;


        await nft.connect(user1).mintToken(proof, {value: ethers.utils.parseEther("5")});
        console.log("nftBalance User1", await nft.balanceOf(user1.address));

        await nft.connect(user1).mintToken(proof, {value: ethers.utils.parseEther("5")});
        console.log("nftBalance User1", await nft.balanceOf(user1.address));
    });
})
```


### Tool Used
Manual




### Mitigation

Transfer the ownership of the entropyGenerator contract to the TraitForgeNFT address.

```
it("Mint next Gen Tokens when the top is reached for a previous Gen supposing that every gen has 2 tokens as max limit", async()=>{                                 
    
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




#
#




## L-02
Out of gas if a user tries to mint in batch multiple NFT via the mintWithBudget() function.


### Functions and Contracts Involved
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L202-L225

mintWithBudget()
TraitForgeNft.sol


### Impact
Assuming that the prices that are set in the contract are the real ones, 
If a user tries to mint, passing 1 ether as value, the mintWithbudget() function will go in revert.
This happens because of the multiple interactions that the function does, and at a certain point, the gas used by the function is not enough to continue the iteration.

 


### Proof Of Concept

```
const {expect} = require("chai");
const {expectRevert} = require("@nomicfoundation/hardhat-network-helpers");
const { ethers } = require('hardhat');
const generateMerkleTree = require('../scripts/genMerkleTreeLib');



describe("Out Of Gas", function(){

    let owner, user1, user2, user3, user4, hacker;
    let Trait, trait, TraitNFT, nft, Airdrop, airdrop, EntropyGenerator, entropyGenerator,
        EntityForging, entityForging, DevFund, devFund, NukeFund, nukeFund, DAOFund, daoFund,
        EntityTrading, entityTrading, merkleInfo;

    const ROUTER_ADDRESS = '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D';


    let proof = [];


    before(async()=>{

        [owner, user1, user2, user3, user4, hacker] = await ethers.getSigners();

        // Deploy TOKEN ERC20
        Trait = await ethers.getContractFactory("Trait");
        trait = await Trait.deploy("TRAIT", "TRT", 18, ethers.utils.parseEther('1000000'));
        await trait.deployed();


        // Deploy DAOFund
        DAOFund = await ethers.getContractFactory('DAOFund');
        daoFund = await DAOFund.deploy(trait.address, ROUTER_ADDRESS);
        await daoFund.deployed();


        // Deploy NFT
        TraitNFT = await ethers.getContractFactory("TraitForgeNft");
        nft = await TraitNFT.deploy();
        await nft.deployed();


        // Deploy Airdrop
        Airdrop = await ethers.getContractFactory('Airdrop');
        airdrop = await Airdrop.deploy();
        await airdrop.deployed();


        // @Audit da controllare
        await nft.setAirdropContract(airdrop.address);
        await airdrop.transferOwnership(nft.address);


        // Deploy EntropyGenerator
        EntropyGenerator = await ethers.getContractFactory('EntropyGenerator');
        entropyGenerator = await EntropyGenerator.deploy(nft.address);
        await entropyGenerator.deployed();

        await entropyGenerator.writeEntropyBatch1();
        await nft.setEntropyGenerator(entropyGenerator.address);


        // Deploy EntityForging contract
        EntityForging = await ethers.getContractFactory('EntityForging');
        entityForging = await EntityForging.deploy(nft.address);
        await entityForging.deployed();

        await nft.setEntityForgingContract(entityForging.address);


        // Deploy DevFund contract
        DevFund = await ethers.getContractFactory('DevFund');
        devFund = await DevFund.deploy();
        await devFund.deployed();


        // Deploy NukeFund contract
        NukeFund = await ethers.getContractFactory('NukeFund');
        nukeFund = await NukeFund.deploy(
          nft.address,
          airdrop.address,
          devFund.address,
          daoFund.address
        );
        await nukeFund.deployed();
        await nft.setNukeFundContract(nukeFund.address);


        // Deploy EntityTrading contract
        EntityTrading = await ethers.getContractFactory('EntityTrading');
        entityTrading = await EntityTrading.deploy(nft.address);
        await entityTrading.deployed();
        await entityTrading.setNukeFundAddress(nukeFund.address);



        // Generate MarkleTree
        merkleInfo = generateMerkleTree([
          owner.address,
          user1.address,
          //user2.address,
          user3.address,
          user4.address,
          hacker.address,
        ]);

        await nft.setRootHash(merkleInfo.rootHash);
    });


    
    it("Mint Out Of Gas", async()=>{

        proof = merkleInfo.whitelist[1].proof;

        await nft.connect(user1).mintWithBudget(proof, {value: ethers.utils.parseEther("1")});

        console.log(await nft.balanceOf(user1.address));
    });                                                                                      
```


### Tool Used
Manual



### Mitigation

Add a maximum amount that is possible to mint for a user.
For example, if every user can mint 5 Tokens max, this function will work properly.

This Mitigation can solve 2 problems:

1) Holder Centralization
If every user can mint as many tokens as want, there is the risk of "centralized holders".
This means that there is the possibility that 10-20 holders, will be the owners of all the NFTs.


2) Function mintWithBudget() will works properly.










#
#





## L-03
Is it possible to set a withdrawEndDate less than the actual timestamp.


### Functions and Contracts Involved
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/TraitForgeNft/TraitForgeNft.sol#L126C3-L128

setWhitelistEndTime()
TraitForgeNft.sol


### Impact
Assuming that the actual withdrawEndDate is 100 and the actual block is 80, if the owner of the TraitForgeNft contract, calls the setWithdrawEndDate() passing 70 as a new end date, there is not a revert, and the mintToken() functions become accessible to anyone.

 


### Proof Of Concept

```
const {expect} = require("chai");
const {expectRevert, time} = require("@nomicfoundation/hardhat-network-helpers");
const { ethers } = require('hardhat');
const generateMerkleTree = require('../scripts/genMerkleTreeLib');



describe("Wrong timestamp Passed", function(){

    let owner, user1, user2, user3, user4, hacker;
    let Trait, trait, TraitNFT, nft, Airdrop, airdrop, EntropyGenerator, entropyGenerator,
        EntityForging, entityForging, DevFund, devFund, NukeFund, nukeFund, DAOFund, daoFund,
        EntityTrading, entityTrading, merkleInfo;

    const ROUTER_ADDRESS = '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D';


    let proof = [];


    before(async()=>{

        [owner, user1, user2, user3, user4, hacker] = await ethers.getSigners();

        // Deploy TOKEN ERC20
        Trait = await ethers.getContractFactory("Trait");
        trait = await Trait.deploy("TRAIT", "TRT", 18, ethers.utils.parseEther('1000000'));
        await trait.deployed();


        // Deploy DAOFund
        DAOFund = await ethers.getContractFactory('DAOFund');
        daoFund = await DAOFund.deploy(trait.address, ROUTER_ADDRESS);
        await daoFund.deployed();


        // Deploy NFT
        TraitNFT = await ethers.getContractFactory("TraitForgeNft");
        nft = await TraitNFT.deploy();
        await nft.deployed();


        // Deploy Airdrop
        Airdrop = await ethers.getContractFactory('Airdrop');
        airdrop = await Airdrop.deploy();
        await airdrop.deployed();


        // @Audit da controllare
        await nft.setAirdropContract(airdrop.address);
        await airdrop.transferOwnership(nft.address);


        // Deploy EntropyGenerator
        EntropyGenerator = await ethers.getContractFactory('EntropyGenerator');
        entropyGenerator = await EntropyGenerator.deploy(nft.address);
        await entropyGenerator.deployed();

        await entropyGenerator.writeEntropyBatch1();
        await nft.setEntropyGenerator(entropyGenerator.address);


        // Deploy EntityForging contract
        EntityForging = await ethers.getContractFactory('EntityForging');
        entityForging = await EntityForging.deploy(nft.address);
        await entityForging.deployed();

        await nft.setEntityForgingContract(entityForging.address);


        // Deploy DevFund contract
        DevFund = await ethers.getContractFactory('DevFund');
        devFund = await DevFund.deploy();
        await devFund.deployed();


        // Deploy NukeFund contract
        NukeFund = await ethers.getContractFactory('NukeFund');
        nukeFund = await NukeFund.deploy(
          nft.address,
          airdrop.address,
          devFund.address,
          daoFund.address
        );
        await nukeFund.deployed();
        await nft.setNukeFundContract(nukeFund.address);


        // Deploy EntityTrading contract
        EntityTrading = await ethers.getContractFactory('EntityTrading');
        entityTrading = await EntityTrading.deploy(nft.address);
        await entityTrading.deployed();
        await entityTrading.setNukeFundAddress(nukeFund.address);



        // Generate MarkleTree
        merkleInfo = generateMerkleTree([
          owner.address,
          user1.address,
          //user2.address,
          user3.address,
          user4.address,
          hacker.address,
        ]);

        await nft.setRootHash(merkleInfo.rootHash);
    });



    it("Set End Whitelist less than the actual timestamp", async()=>{
        proof = merkleInfo.whitelist[1].proof;
        await nft.connect(user1).mintToken(proof, {value: ethers.utils.parseEther("2")});

        const actualBlock = await ethers.provider.getBlock("latest");
        const timing = actualBlock.timestamp;


        // Owner set the wrong timestamp
        await nft.setWhitelistEndTime(timing - 10);

        await nft.connect(user2).mintToken(["0x0000000000000000000000000000000000000000000000000000000000000000"], {value: ethers.utils.parseEther("2")});
    });
})                                                                                     
```




### Mitigation

Modify the setWhitelistEndTime() function, checking that the new endtime 
is not passed yet.


```
function setWhitelistEndTime(uint256 endTime_) external onlyOwner {
    require(endtime_ > whitelistEndTime, "Wrong end time!");
    whitelistEndTime = endTime_;
  }
```


#
#


## L-04 
Set a check that reverts in case address 0 is set on these functions:


### EntityForging.sol:
1. function setNukeFundAddress(address payable _nukeFundAddress);

### EntityTrading.sol
2. function setNukeFundAddress(address payable _nukeFundAddress);

### EntropyGenerator.sol
3. function setAllowedCaller(address _allowedCaller);


### NukeFund.sol
4. function setAirdropContract(address _airdrop);
5. function setTraitForgeNftContract(address _traitForgeNft);
6. function setDevAddress(address payable account);
7. function setDaoAddress(address payable account);


### TraitForgeNft.sol
8. function setNukeFundContract(address payable _nukeFundAddress);
9. function setEntityForgingContract(address entityForgingAddress_);
10. function setEntropyGenerator(address entropyGeneratorAddress_);
11. function setAirdropContract(address airdrop_);


### Impact
It is possible, that some interactions, can send funds to address 0.


### Tool Used
Manual




### Mitigation

An example of mitigation of one of the functions highlighted

```
function setNukeFundContract(
    address payable _nukeFundAddress
  ) external onlyOwner {
    require(_nukeFundAddress != address(0), "Address 0 setted!");
    nukeFundAddress = _nukeFundAddress;
    emit NukeFundContractUpdated(_nukeFundAddress); 
  }                                                                                         
```


#
#



## L-05 
Set a check that reverts in case a 0 value is set on these functions:


### EntityForging.sol:
1. function setTaxCut(uint256 _taxCut);
2. function setOneYearInDays(uint256 value);
3. function setMinimumListingFee(uint256 _fee);
. 

### EntityTrading.sol
4. function setTaxCut(uint256 _taxCut)


### NukeFund.sol
5. function setTaxCut(uint256 _taxCut);
6. function setMinimumDaysHeld(uint256 value);
7. function setDefaultNukeFactorIncrease(uint256 value);
8. function setMaxAllowedClaimDivisor(uint256 value);
9. function setNukeFactorMaxParam(uint256 value);
10. function setAgeMultplier(uint256 _ageMultiplier);


### TraitForgeNft.sol
11. function setStartPrice(uint256 _startPrice);
12. function setPriceIncrement(uint256 _priceIncrement);
13. function setPriceIncrementByGen(uint256 _priceIncrementByGen);



### Impact
It is possible, that some calculations, go in revert or return wrong results if there is a 0 set. 


### Tool Used
Manual




### Mitigation

An example of mitigation of one of the functions highlighted.
This variable, taxCut, is used in some of the main calculations, and most of the time is the dividend of the operations.
If is set to 0, will go in revert because division by 0 is impossible. 

```
function setTaxCut(uint256 _taxCut) external onlyOwner {
    require(_taxCut != 0, "Impossible set 0!");
    taxCut = _taxCut;
  }                                                                                     
```







