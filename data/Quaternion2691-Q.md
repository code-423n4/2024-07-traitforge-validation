### [L-1] Pushing funds directly to forger accounts may waste gas if user contract reverts on receive

**Description:** 
Whenever a merger token owner calls `EntityForging::forgeWithListed` the forgerShare is directly sent to the forger. This could revert the call as user may use contract and has receive function revert with error. 
[Link](https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntityForging/EntityForging.sol#L158)

**Impact:** 
The impact of this is low, since user will be wasting small amount of gas since reverts are not for free. 

**Recommended Mitigation:** 
1. Allow push over pull mechanism for allowing forgers to withdraw funds from smart contract. 
```diff
contract EntityForging is IEntityForging, ReentrancyGuard, Ownable, Pausable {
.
.
.
+   mapping(address forgerAddress => uint256 forgerFeesCollected) private s_forger_to_fees_accrued;
.
.
.
    function forgeWithListed( uint256 forgerTokenId, uint256 mergerTokenId){
.
.
.
-       (bool success_forge, ) = forgerOwner.call{ value: forgerShare }('');
-       require(success_forge, 'Failed to send to Forge Owner');      
+       s_forger_to_fees_accrued[forgeOwner] += forgerShare;   
    }
         
+   function withdrawFees() public payable {
+       uint256 amount = s_forger_to_fees_accrued[msg.sender]; 
+       s_forger_to_fees_accrued[forgeOwner] = 0; 
+       (bool success, ) = payable(msg.sender).call{value: amount}(""); 
+       require(success_forge, 'Eth withdrawal failed');
    }
.
.
.
}
``` 

## [L-2] `EntropyGenerator::deriveTokenParameters` outputs wrong nukeFactor value

**Lines of code**

https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L152

**Description**
The `EntropyGenerator::deriveTokenParameters` wrongly informs the users of the nuking potential of Nfts by always giving nuking factor as 0. 

**Impact**
This would incorrectly inform the users of their initialNukeFactor, and eventually the potential amount of funds they can accrue in the nuking of NFT.

**Proof of Concept**
paste the following code in describe test of `EntropyGenerator.test.ts` 
```javascript
  it("should check that nuke factor is always zero", async function(){
    let cnt = 0 ;
    for(let slotIndex = 0 ; slotIndex<700 ; slotIndex++){
      for(let numberIndex = 0; numberIndex<12 ; numberIndex++ ){
        const [nukeFactor, x, y, z] = await entropyGenerator.deriveTokenParameters(slotIndex, numberIndex);
        if(nukeFactor == 0n) cnt++;   
      }
    }
    console.log(cnt); 
  })
```
The count here will be 8400

**Tools Used**
Manual Review, Hardhat Test

**Recommended Mitigation Steps**
1. correct the number of trailing zeroes while calculating the nukeFactor.
```diff
    function deriveTokenParameters(
    uint256 slotIndex,
    uint256 numberIndex
  )
    public
    view
    returns (
      uint256 nukeFactor,
      uint256 forgePotential,
      uint256 performanceFactor,
      bool isForger
    )
  {
    uint256 entropy = getEntropy(slotIndex, numberIndex);

    // example calcualtions using entropyto derive game-related parameters
-    nukeFactor = entropy / 4000000;
+    nukeFactor = entropy / 40; 
.
.
.
    return (nukeFactor, forgePotential, performanceFactor, isForger); 
  }
```
**Assessed type**

Math