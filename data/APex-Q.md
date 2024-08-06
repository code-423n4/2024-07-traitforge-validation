## [L-1] `EntityForging::nukeFundAddress` is unnecessarily marked as payable. 

**Description:** The `EntityForging::nukeFundAddress` is marked as payable address but the purpose is unclear. If it's not correctly initialized, it can lead to failed transactions. Additionally the `EntityForging::setNukeFundAddress` function is also marked as payable. This is unneeded. 

```javascript
//@audit 
- address payable public nukeFundAddress;
+ address public nukeFundAddress;
...

 function setNukeFundAddress( 
-    address payable _nukeFundAddress
+    address _nukeFundAddress
  ) external onlyOwner {
    nukeFundAddress = _nukeFundAddress;
  }
```
