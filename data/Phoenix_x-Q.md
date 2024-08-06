## Vulnerability details

The method `EntropyGenerator::getNextEntropy` is used for generating entropies and it must revert when the maximum amount of entropies is reached - 10000.
However, it does not do that after the 10000 time and it continues emiting value.


## Proof of concept:
- Add the following test case and observe that it emits value and the check `require(slotIndex <= maxSlotIndex, 'Slot index out of bounds.');` does not trigger after the 10000 time.

```javascript
it('test-entropy', async function () {
  for (let i = 0; i < 10002; i++) {
    await expect(
      entropyGenerator.connect(allowedCaller).getNextEntropy()
    ).to.emit(entropyGenerator, 'EntropyRetrieved');
  }
});
```

- Run the following command to run the test `yarn test` and keep in mind that the way this test is written will take time to execute.

- Verify that the test passes to prove that an event is emited for generated entropy after the 10000 time.