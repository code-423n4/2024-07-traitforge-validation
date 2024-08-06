1. The listedTokenIds variable was not cleared properly.
In the EntityForging and EntityTrading contracts, the data in listedTokenIds is not deleted when the list is canceled or the list is executed.