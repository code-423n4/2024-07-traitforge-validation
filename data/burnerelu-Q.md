## Summary

Current nuking mechanism creates big incentive to nuke as soon as possible

## Details

Let's assume the following:
User X is the first minter; he deposits 0.005 ETH, and mints the average Entity, with entropy 555555. For an `ageMultiplier` set to 1, after the 3 days have passed, he would have a nuke factor of 13989, which is around 14%.

Let's initially assume the first minter is the only minter in his block, and therefore has the luxury of deciding the first move.

Taking the 10% tax into account and using the above data, once 8 players have signed up the initial player would already obtain a small profit for nuking. If a 10th player mints, it would already be profitable to nuke and mint a new NFT hoping for better stats.
Once 100 players sign up, nuking would fetch sufficient money to buy 10 new NFTs. 
After 250 players sign up, user X would already afford to buy the last NFT from gen. 1. After this nuke, the pot would lose the contributions of the last 26 players. It would take 24 new mints to bring the pot to the same value as before.

A nuke with 1000 active players would remove the contribution of ~86 players, and would need ~79 new mints to bring the pot back to the same value. The difference in income from nuking first and second is more than 0.3 ETH, where the difference between the first and the 10th is more than 1.6 ETH.

Now let's assume there are more minting transactions in the initial block. There is an even bigger incentive to leverage your position and maximize the profits, especially since it's still possible to join the game.

If the game is already full, the first minter(s) would fight over an initial nuke of around ~160 ETH, while the second for ~138 ETH. The 19th would still get over 10 ETH.

## Proposed solution

The incentive to nuke early could be downplayed by a randomized failure factor. This factor could produce a chance to bring a smaller yield for a less mature entity (and smaller generations), and therefore incentivize the user to wait as much as possible until breaking the bank. 


