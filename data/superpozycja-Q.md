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
		vm.roll(1);
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
  for digit  6
  0 12425
  1 4320
  2 4025
  3 4170
  4 4060
  5 4055
  6 4470
  7 4050
  8 4290
  9 4180
  for digit  5
  0 5670
  1 4975
  2 5045
  3 5095
  4 4910
  5 4915
  6 4845
  7 4915
  8 4835
  9 4840
  for digit  4
  0 5090
  1 5015
  2 5130
  3 4965
  4 5165
  5 5070
  6 4775
  7 5005
  8 4820
  9 5010
  for digit  3
  0 4805
  1 4995
  2 4980
  3 5075
  4 5170
  5 4780
  6 5255
  7 5110
  8 5120
  9 4755
  for digit  2
  0 5050
  1 4905
  2 4935
  3 5270
  4 5215
  5 5020
  6 4650
  7 5040
  8 5190
  9 4770
  for digit  1
  0 0
  1 5710
  2 5805
  3 5955
  4 5450
  5 5610
  6 5505
  7 5340
  8 5330
  9 5340
```

Visualized:
![image](https://gist.github.com/user-attachments/assets/baeb8bae-09f7-42ac-95ac-526ee4abec57)

It's worth noting: the distribution of the first digit is particularly important for the contract logic because combining this issue with L-01 means that `forgePotential` will never be `0`, which is in direct conflict with [the documentation](https://docs.google.com/document/d/1pihtkKyyxobFWdaNU4YfAy56Q7WIMbFJjSHUAfRm6BA/edit) which does assume the existence of infertile (`forgePotential == 0`) entities.

The missing `0` in the distribution for digit 1 can be fixed by not using the padded entropy all together. Replacing `paddedEntropy` with `entropy`, though, generates the following distribution for digit 1:
```
  for digit  1
  0 8320
  1 4575
  2 5090
  3 4660
  4 4545
  5 4450
  6 4445
  7 4530
  8 4725
  9 4705
```

suggesting there might be some other flaw in the entropy generator.

### Suggested resolution
Return `entropy` instead of `paddedEntropy` and update `getFirstDigit()` to accomodate for fixed digit numbers (meaning, ensure that `getFirstDigit(000001) == 0`).

## L-03: `require(pseudoRandomValue != 999999)` check does not make sense in this context

`pseudoRandomValue` inside the `writeEntropyBatch*()` functions stores all 13 entropies generated at once using the hash function. Checking the entire `uint256` slot against the God entity's entropy value is incorrect.

Offending code:
https://github.com/code-423n4/2024-07-traitforge/blob/279b2887e3d38bc219a05d332cbcb0655b2dc644/contracts/EntropyGenerator/EntropyGenerator.sol#L46-L98

Iterate over entropy numbers in a slot in an inner loop and check the requirement correctly one value at a time.