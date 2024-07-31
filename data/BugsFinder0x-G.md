## Title
Is it possible to set a withdrawEndDate less than the actual timestamp.


## Functions and Contracts Involved
setWhitelistEndTime()
TraitForgeNft.sol


## Impact
Assuming that the actual withdrawEndDate is 100 and the actual block is 80, if the owner of the TraitForgeNft contract, calls the setWithdrawEndDate passing 70 as a new end date, there is not a revert, and the mintToken() functions become accessible to anyone.

 


## Proof Of Concept

```
const {expect} = require("chai");
const {expectRevert, time} = require("@nomicfoundation/hardhat-network-helpers");
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



    it("Set End Whitelist less than the actual timestamp", async()=>{
        proof = merkleInfo.whitelist[1].proof;
        await nft.connect(user1).mintToken(proof, {value: ethers.utils.parseEther("2")});

        const actualBlock = await ethers.provider.getBlock("latest");
        const timing = actualBlock.timestamp;


        // Owner set wrong timestamp
        await nft.setWhitelistEndTime(timing - 10);

        await nft.connect(user2).mintToken(["0x0000000000000000000000000000000000000000000000000000000000000000"], {value: ethers.utils.parseEther("2")});
    });
})                                                                                     
```




## Mitigation

Modify the setWhitelistEndTime() function, checking that the new endtime is not passed yet.

```
function setWhitelistEndTime(uint256 endTime_) external onlyOwner {
    require(endtime_ > whitelistEndTime, "Wrong end time!");
    whitelistEndTime = endTime_;
  }
```