EntityForging has a setter function called setOneYearInDays, which can change the value of the oneYearInDays state variable. But this is completely unnecessary, since oneYearInDays never changes, and should be a constant.

Recommendation:
Make oneYearInDays a constance and remove the setter function.

EntityForging fetchListings function will stop working once there are too many listings, since it will eventually run out of gas. 

Recommendation:
Instead of trying to get all listings, you should allow "pagination", ie. let the caller specify a start and end to the positions of the listings to query.