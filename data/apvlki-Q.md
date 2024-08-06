### `writeEntropyBatch3()` can be triggered publicly anyone, so anyone can possibly mint the golden god attributes before the intended use

We should also add a protection atleast to `writeEntropyBatch3()` to have more control on when the golden god attributes to appear. 

```
// the functions can be called right after each other
entropyGenerator.writeEntropyBatch1();
entropyGenerator.writeEntropyBatch2();
// the restriction opens after this function call
entropyGenerator.writeEntropyBatch3();
```





