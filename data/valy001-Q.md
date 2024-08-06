It is recommended to delete items in listedTokenIds as well as in listings in function _cancelListingForForging() in contract EntityForging.sol.

delete listings[listedTokenIds[tokenId]] solely would cause the two mapping $listedTokenIds$ and $listings$ no longer holding index-item projection， which may cause confuse when users call get function to getListedTokenIds(uint tokenId_) and getListings(uint id), as the former will return an uint index number(>0), while the later will return an empty $Listing$. 