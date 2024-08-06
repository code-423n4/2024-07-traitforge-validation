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