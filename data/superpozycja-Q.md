## L-01: `forgePotential` is calculated incorrectly
Forge Potential is said to be calculated from the fifth digit (counting from the left) in the base10 representation of the entropy - according to [the protocol's website](https://traitforge.info/howitworks) and [the whitepaper](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit).

However, the code doesn't reflect that, as the `forgePotential` variable is assigned the result of `getFirstDigit(entropy)`.

### Offending code
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L135-L161

### Tests
We can compare and check that this mismatch truly happens:

```Solidity
function testForgePotential() public {
	address tnft = makeAddr("traitforge");
	vm.startPrank(tnft);
	EntropyGenerator e = new EntropyGenerator(tnft);
	e.writeEntropyBatch1();
	e.writeEntropyBatch2();
	e.writeEntropyBatch3();
	vm.stopPrank();
	uint256 ent = e.getPublicEntropy(100, 10); /* arbitrary indices; can be anything really */
	console.log("entropy value: ", ent);
	ent /= 10;
	ent = ent%10; /* get the fifth digit from the left */
	console.log("expected forgePotential value: ", ent);
	uint256 fp;
	(, fp, , ) = e.deriveTokenParameters(100, 10);
	console.log("actual forgePotential value: ", fp);
	assertEq(ent, fp);
}

```
```
2024-07-traitforge [main] $ forge test -vv
[⠊] Compiling...
No files changed, compilation skipped

Ran 1 test for test/EntropyPwner.t.sol:EntropyPwnerTest
[FAIL. Reason: assertion failed: 1 != 6] testForgePotential() (gas: 18270600)
Logs:
  entropy value:  621219
  expected forgePotential value:  1
  actual forgePotential value:  6

Suite result: FAILED. 0 passed; 1 failed; 0 skipped; finished in 2.32ms (2.09ms CPU time)

Ran 1 test suite in 3.60ms (2.32ms CPU time): 0 tests passed, 1 failed, 0 skipped (1 total tests)

Failing tests:
Encountered 1 failing test in test/EntropyPwner.t.sol:EntropyPwnerTest
[FAIL. Reason: assertion failed: 1 != 6] testForgePotential() (gas: 18270600)

Encountered a total of 1 failing tests, 0 tests succeeded
```

### Suggested resolution
Update code logic to correctly calculate the fifth digit of the entropy number.  

## L-02: Entropy padding calculation makes it so the entropy distribution is not uniform

Running a simple statistical test shows that the entropy distribution has a flaw, resulting in a non-uniform distribution of generated digits. This is caused by the logic in `getEntropy()`, more specifically by the padding calculation. As it is now, the padding *shifts* the input number so that it becomes a 6-digit number starting with a non-zero digit. This effectively skews the entropy number distribution, preventing any number starting with 0 from appearing.

### Offending code
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L163-L185

### Tests
Test ran to generate the data:
```Solidity
function getDigit(uint256 n, uint256 d) public returns (uint256) {
	for (uint256 i = 0; i < d; i++) {
		n /= 10;
	}
	return n % 10;
}

function testPRNG() public {
	address vic = makeAddr("victim");
	address att = makeAddr("attacker");


	uint256[10][6] memory distribution;

	for (uint256 iters = 0; iters < 5; iters++) {
		vm.roll(iters+1);
		vm.startPrank(vic);
		EntropyGenerator e = new EntropyGenerator(vic);
		e.writeEntropyBatch1();
		e.writeEntropyBatch2();
		e.writeEntropyBatch3();
		vm.stopPrank();

		vm.startPrank(att);
		for (uint256 si = 0; si < 770; si++) { 
			for (uint256 ni = 0; ni < 13; ni++) {
				if ((si+1) * (ni+1) <= 10000) {
					for (uint256 i = 0; i < 6; i++) {
						uint256 a = e.getPublicEntropy(si, ni);
						distribution[i][getDigit(a, i)] += 1;
					}
				}
			}
		}
	}

	for (uint256 i = 0; i < 6; i++) {
		console.log("for digit ", 6-i);
		for (uint256 j = 0; j < 10; j++) {
			console.log(j, distribution[i][j]);
		}
	}
	vm.stopPrank();

}
```
Running this results in:
```
Logs:
Logs:
  for digit  6
  0 12572
  1 4206
  2 4081
  3 4140
  4 4172
  5 4146
  6 4281
  7 4169
  8 4048
  9 4230
  for digit  5
  0 5878
  1 4881
  2 4911
  3 4976
  4 4938
  5 4949
  6 4978
  7 4837
  8 4866
  9 4831
  for digit  4
  0 5155
  1 5070
  2 4867
  3 4941
  4 5086
  5 4974
  6 4982
  7 4979
  8 4906
  9 5085
  for digit  3
  0 4945
  1 4979
  2 5091
  3 5059
  4 5009
  5 4949
  6 5021
  7 4988
  8 5072
  9 4932
  for digit  2
  0 5001
  1 5061
  2 5030
  3 5090
  4 5133
  5 4943
  6 4950
  7 4951
  8 4995
  9 4891
  for digit  1
  0 0
  1 5910
  2 5940
  3 5866
  4 5606
  5 5468
  6 5523
  7 5260
  8 5209
  9 5263
```

Visualized:
[Link to image](https://gist.github.com/user-attachments/assets/baeb8bae-09f7-42ac-95ac-526ee4abec57)

It's worth noting: the distribution of the first digit is particularly important for the contract logic because combining this issue with L-01 means that `forgePotential` will never be `0`, which is in direct conflict with [the documentation](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit) which does assume the existence of infertile (`forgePotential == 0`) entities.

The missing `0` in the distribution for digit 1 can be fixed by not using the padded entropy all together. Replacing `paddedEntropy` with `entropy`, though, generates the following distribution for digit 1:
```
  for digit  1
  0 8489
  1 4671
  2 4669
  3 4607
  4 4572
  5 4531
  6 4609
  7 4571
  8 4638
  9 4688
```

suggesting there might be some other flaw in the entropy generator.

### Suggested resolution
Return `entropy` instead of `paddedEntropy` and update `getFirstDigit()` to accomodate for fixed digit numbers (meaning, ensure that `getFirstDigit(000001) == 0`).

## L-03: `require(pseudoRandomValue != 999999)` check does not make sense in this context

`pseudoRandomValue` inside the `writeEntropyBatch*()` functions stores all 13 entropies generated at once using the hash function. Checking the entire `uint256` slot against the God entity's entropy value is incorrect.

### Offending code:
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L46-L98

### Suggested resolution
Iterate over entropy numbers in a slot in an inner loop and check the requirement correctly one value at a time.