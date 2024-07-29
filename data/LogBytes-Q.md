# Wrong total supply minted when deploying the contract

Although the issue is out of scope, we would like to mention an issue with the deployment script setting the wrong token total supply. 


When reading the [Airdrop documentation]("https://github.com/TraitForge/GitBook/blob/main/GamePlay/%24TRAIT%20Token%20Airdrop.md") the team mentions a total supply of 1 billion tokens, but when deploying the token the wrong total supply is minted.

## **Impact**

The TRAIT token does not implement a mint function so after deployment the totalSupply can’t be incremented. Which results in a DOS of the Airdrop.sol contract as the token amounts can’t be transferred to the contract because it exceeds the total supply. Protocol need to redeploy the token contract again.

## **Proof of concept**

There is a task deploy-token which deploys the TRAIT token, but the totalSupply is set to 1 million instead of 1 billion.

1_000_000 = 1 million 
1_000_000_000 = 1 billion

```
task('deploy-token', 'Deploy Trait Token').setAction(async (_, hre) => {
  const name = 'TRAIT';
  const symbol = 'TRAIT';
  const decimals = 18;
  const totalSupply = ethers.parseEther('1000000'); // 1_000_000 must be 1_000_000_000

  try {
    console.log('Deploying Trait...');
    const token = await hre.ethers.deployContract('Trait', [
      name,
      symbol,
      decimals,
      totalSupply,
    ]);
    await token.waitForDeployment();
    console.log('Contract deployed to:', await token.getAddress());
    return token;
  } catch (error) {
    console.error(error);
  }
  return null;
});

```
Create a new test `poc.test.ts`

`npx hardhat test test/poc.test.ts`

totalSupply does not match the 1 billion.

```tsx
import { HardhatEthersSigner } from '@nomicfoundation/hardhat-ethers/signers';
import { expect } from 'chai';
import { ethers } from 'hardhat';
import { Trait } from '../typechain-types';

describe('POC:', function () {
  let owner: HardhatEthersSigner;
  let user1: HardhatEthersSigner;
  let user2: HardhatEthersSigner;
  let token: Trait;

  before(async () => {
    [owner, user1, user2] = await ethers.getSigners();

    token = await ethers.deployContract('Trait', [
      'Trait',
      'TRAIT',
      18,
      ethers.parseEther('1000000'), // 1 million 1000000 vs 1000000000 
    ]);
  });

  it('should succeed: does have 1 billion total supply', async () => {
    const expectedTotalSupply = ethers.parseEther('1000000000'); // 1_000_000_000
    const totalSupply = await token.totalSupply();
    expect(totalSupply).to.eq(expectedTotalSupply);
  });
});

```

## **Recommended mitigation steps**

Adding three missing zero's to make 1 billion total supply

```diff
task('deploy-token', 'Deploy Trait Token').setAction(async (_, hre) => {
  const name = 'TRAIT';
  const symbol = 'TRAIT';
  const decimals = 18;
-  const totalSupply = ethers.parseEther('1000000');
+  const totalSupply = ethers.parseEther('1000000000'); // add three missing zero's
  try {
    console.log('Deploying Trait...');
    const token = await hre.ethers.deployContract('Trait', [
      name,
      symbol,
      decimals,
      totalSupply,
    ]);
    await token.waitForDeployment();
    console.log('Contract deployed to:', await token.getAddress());
    return token;
  } catch (error) {
    console.error(error);
  }
  return null;
});

```


```
npx hardhat test test/poc.test.ts
```


```
POC:
    ✔ should succeed: does have 1 billion total supply
  1 passing (335ms)

```