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


# Swap function not correctly implemented in DaoFund.sol

Although the contract is out of scope. we would like to mention it.

The `swapExactETHForTokens` in the receive function in the `DaoFund.sol` contract is not correctly implemented and is missing slippage control or deadline. This makes the function vulnerable to sandwich attacks and MEV exploits leading to loss of tokens when swapping.

## Proof of Concept

The [Uniswap V2 documentation](https://docs.uniswap.org/contracts/v2/reference/smart-contracts/router-02#swapexactethfortokens) explains that `amountOutMin` is the minimum amount of output tokens that must be received for the transaction not to revert. If this is set to 0 as in the DaoFund.sol contract it becomes vulnerable to [sandwich attacks](https://medium.com/coinmonks/defi-sandwich-attack-explain-776f6f43b2fd) leading to loss of tokens when swapping.

```
uniswapV2Router.swapExactETHForTokens{ value: msg.value }(
   0,// should be never 0
   path,
   address(this),
   block.timestamp // should be + time
);
```

## **Recommended mitigation steps**

the `amountOutMin` must be specified by the user and result must be compared to the tokens received in the contract.

```diff
-receive() external payable {
+receive(uint256 _minAmountOut) external payable {
    require(msg.value > 0, 'No ETH sent');
+   require(_minAmountOut > 0, '_minAmountOut must be > 0');

    address[] memory path = new address[](2);
    path[0] = uniswapV2Router.WETH();
    path[1] = address(token);

 uniswapV2Router.swapExactETHForTokens{ value: msg.value }(
-      0,
+     _minAmountOut,
      path,
      address(this),
-      block.timestamp
+.    block.timestamp + 1 hours
    );
    
+   require(token.balanceOf(address(this)) >= _minAmountOut, 'No slippage');

    require(
      token.burn(token.balanceOf(address(this))) == true,
      'Token burn failed'
    );
  }
```


# User claim methods should not have whenNotPaused modifier

Although the contract is out of scope, we would like to mention it.

Using the `whenNotPaused` modifier on methods where users can claim tokens/rewards/yields shouldn’t have the `whenNotPaused` modifier. It opens up an attack vector where protocol owner can decides if users are able to claim funds from the contract.  There is also the possibility that an admin pauses the contracts and renounces ownership, which will leave the funds stuck in the contract forever.

## Link to code
https://github.com/code-423n4/2024-07-traitforge/blob/main/contracts/Airdrop/Airdrop.sol#L67-L74

## Recommendation mitigation steps

Since the function allows users to claim their airdrop rewards, funds must be claimable anytime.

Users should be able to withdraw their tokens or claim their airdrop tokens even though the contract is in a paused state by not using the `whenNotPaused` modifier on the claim function. 

[X post mentioning vulnerability](https://x.com/chrisdior777/status/1817287618645659971?t=R2akF6RXfZE2PWBjV26OYg&s=19)

```diff
- function claim() external whenNotPaused nonReentrant {
+ function claim() external nonReentrant {
    require(started, 'Not started');
    require(userInfo[msg.sender] > 0, 'Not eligible');

    uint256 amount = (totalTokenAmount * userInfo[msg.sender]) / totalValue;
    traitToken.transfer(msg.sender, amount);
    userInfo[msg.sender] = 0;
  }
```