##  Functionality for _pause/_unpause not existing
## Impact
 The absence of pause/unpause functionality means the contract cannot be paused in 
 emergencies or during critical updates, leaving it vulnerable to potential 
 exploits or operational issues.
## Description
 The contract is implementing the `whenNotPaused` modifier throughout all the the 
 functions that can be trigered by the user but there is no functionality that 
 allows anyone to _pause or _unpause theses functions in an emergency
## Recommendation
 To fully utilize the whenNotPaused modifier and enable the ability to manage the contract's state
```javascript
function pause() public onlyOwner {
    _pause();
}

function unpause() public onlyOwner {
    _unpause();
}

```
