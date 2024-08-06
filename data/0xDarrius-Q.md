# TraitForgeNft and EntityForging contracts cannot receive ETH

The TraitForgeNft and EntityForging contracts have functions that require the user to send eth to pay fees or mint NFTs. But users risk doing this without needing to pay in ETH because these contracts have no receiver or fallback function to receive eth, which doesn't allow the project to achieve its objectives for the NukeFund contract.

## POC

### 1 EntityForging

```$
import {
  Airdrop,
  DevFund,
  EntityForging,
  EntropyGenerator,
  EntityTrading,
  NukeFund,
  TraitForgeNft,
} from '../../typechain-types';
import { HardhatEthersSigner } from '@nomicfoundation/hardhat-ethers/signers';
import generateMerkleTree from '../../scripts/genMerkleTreeLib';

const { expect } = require('chai');
const { ethers } = require('hardhat');

describe('EntityForging', () => {
  let entityForging: EntityForging;
  let nft: TraitForgeNft;
  let owner: HardhatEthersSigner;
  let user1: HardhatEthersSigner;
  let user2: HardhatEthersSigner;
  let user3: HardhatEthersSigner;
  let entityTrading: EntityTrading;
  let nukeFund: NukeFund;
  let devFund: DevFund;
  let FORGER_TOKEN_ID: number;
  let MERGER_TOKEN_ID: number;

  const FORGING_FEE = ethers.parseEther('1.0'); // 1 ETH

  before(async () => {
    [owner, user1, user2, user3] = await ethers.getSigners();

    // Deploy TraitForgeNft contract
    const TraitForgeNft = await ethers.getContractFactory('TraitForgeNft');
    nft = (await TraitForgeNft.deploy()) as TraitForgeNft;

    // Deploy Airdrop contract
    const airdropFactory = await ethers.getContractFactory('Airdrop');
    const airdrop = (await airdropFactory.deploy()) as Airdrop;

    await nft.setAirdropContract(await airdrop.getAddress());

    await airdrop.transferOwnership(await nft.getAddress());

    // Deploy EntityForging contract
    const EntropyGenerator = await ethers.getContractFactory(
      'EntropyGenerator'
    );
    const entropyGenerator = (await EntropyGenerator.deploy(
      await nft.getAddress()
    )) as EntropyGenerator;

    await entropyGenerator.writeEntropyBatch1();

    await nft.setEntropyGenerator(await entropyGenerator.getAddress());

    // Deploy EntityForging contract
    const EntityForging = await ethers.getContractFactory('EntityForging');
    entityForging = (await EntityForging.deploy(
      await nft.getAddress()
    )) as EntityForging;
    await nft.setEntityForgingContract(await entityForging.getAddress());

    devFund = await ethers.deployContract('DevFund');
    await devFund.waitForDeployment();

    const NukeFund = await ethers.getContractFactory('NukeFund');

    nukeFund = (await NukeFund.deploy(
      await nft.getAddress(),
      await airdrop.getAddress(),
      await devFund.getAddress(),
      owner.address
    )) as NukeFund;
    await nukeFund.waitForDeployment();

    await nft.setNukeFundContract(await nukeFund.getAddress());

    entityTrading = await ethers.deployContract('EntityTrading', [
      await nft.getAddress(),
    ]);

    // Set NukeFund address
    await entityTrading.setNukeFundAddress(await nukeFund.getAddress());

    const merkleInfo = generateMerkleTree([
      owner.address,
      user1.address,
      user2.address,
    ]);

    await nft.setRootHash(merkleInfo.rootHash);

    // Mint some tokens for testing
    await nft.connect(owner).mintToken(merkleInfo.whitelist[0].proof, {
      value: ethers.parseEther('1'),
    });
    await nft.connect(user1).mintToken(merkleInfo.whitelist[1].proof, {
      value: ethers.parseEther('1'),
    });
    await nft.connect(user1).mintToken(merkleInfo.whitelist[1].proof, {
      value: ethers.parseEther('1'),
    });

    for (let i = 0; i < 10; i++) {
      await nft.connect(owner).mintToken(merkleInfo.whitelist[0].proof, {
        value: ethers.parseEther('1'),
      });
      const isForger = await nft.isForger(i + 4);
      if (isForger) {
        FORGER_TOKEN_ID = i + 4;
        break;
      }
    }

    MERGER_TOKEN_ID = 3;
    const test = await ethers.provider.getStorageAt(await nft.getAddress(), 0);

    //console.log(await nft.ownerOf(FORGER_TOKEN_ID));
    //console.log(await nft.ownerOf(MERGER_TOKEN_ID));
    //console.log(user1.address);
    console.log(test, 'test');
    //console.log(await entityForging.forgingCounts(FORGER_TOKEN_ID));
    //console.log(await nft.getTokenEntropy(FORGER_TOKEN_ID));
  });

  describe('listForForging', () => {
    it('should allow the owner to list a token for forging', async () => {
      const tokenId = FORGER_TOKEN_ID;
      const fee = FORGING_FEE;

      await entityForging.connect(owner).listForForging(tokenId, fee);
      await entityForging.listedTokenIds(tokenId);
      await entityForging
        .connect(user1)
        .forgeWithListed(tokenId, MERGER_TOKEN_ID, { value: fee });

      const balance = await ethers.provider.getBalance(
        await entityForging.getAddress()
      );
      expect(balance).to.equal(ethers.parseEther('1.0'));
    });
  });

  describe('send ETH', () => {
    it('Should receive eth in contract', async function () {
      const tx = await owner.sendTransaction({
        to: await entityForging.getAddress(),
        value: ethers.parseEther('1.0'),
      });
      await tx.wait();
      const balance = await ethers.provider.getBalance(
        await entityForging.getAddress()
      );
      expect(balance).to.equal(ethers.parseEther('1.0'));
    });
  });
});
```

### 2. TraitForgeNft

```$
import { TraitForgeNft, EntityForging } from '../../typechain-types';
import { HardhatEthersSigner } from '@nomicfoundation/hardhat-ethers/signers';
import generateMerkleTree from '../../scripts/genMerkleTreeLib';

const { expect } = require('chai');
const { ethers } = require('hardhat');

describe('send ETH', () => {
  let nft: TraitForgeNft;
  let owner: HardhatEthersSigner;
  let user1: HardhatEthersSigner;
  before(async () => {
    [owner, user1] = await ethers.getSigners();

    // Deploy TraitForgeNft contract
    const TraitForgeNft = await ethers.getContractFactory('TraitForgeNft');
    nft = (await TraitForgeNft.deploy()) as TraitForgeNft;
  });
  it('Should receive eth in contract', async function () {
    const tx = await owner.sendTransaction({
      to: await nft.getAddress(),
      value: ethers.parseEther('1.0'),
    });
    await tx.wait();
    const balance = await ethers.provider.getBalance(await nft.getAddress());
    expect(balance).to.equal(ethers.parseEther('1.0'));
  });
});
```

## Recommendation

To solve this problem, just add the receiver function to the 2 contracts.