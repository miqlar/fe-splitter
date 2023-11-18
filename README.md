# fe-splitter



Compile contracts:
```
fe build . --overwrite
```


## Issues/Difficulties faced

- For some reason, "use erc20_token::ERC20" would only work well if i did it from main.fe. If it was from any other file, it would fail and claim the file was not found.
- Documentation missing some important details:
    - To find how to check the address of the contract (ctx.self_address()), I had to dig into the release logs, as I could not find info anywhere.
    - No info of ctx.block_timestamp() either, guessed that one seeing ctx.block_number().
- Tests not working well, right now if you do "fe test", it says 0 tests passed, and 0 failed. Though, there is one in 'test_error.fe'. I attempted to add tests in the main file, but did not manege to make them work. Issue is that if you add the 'self' in get_42(self), it does not run the test, and if you remove it, it complains that "function does not take self".
- (Maybe i did not look too much into this one) - Defining a for loop seems only to be possible when iterating through an array, which is less convenient than the standard C/Solidity for.

##

```
cast send --rpc-url localhost:8545 --private-key <private-key> --create $(cat output/Splitter/Splitter.bin) $(cast abi-encode "__init__(bool,uint256)" false 0)
```

