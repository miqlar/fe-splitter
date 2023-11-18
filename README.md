# fe-splitter

Simple but powerful native balance and ERC20 splitter

- Supports native assets (ETH...)
- Supports ERC20
- Onwer (deployer) can:
    - add N addresses
    - remove addresses
    - modify split share for each address after setting it
- Supports unequal splits
- Has optional Locks
    - Time lock based on block_timestamp
    - Owner lock (so that owner cant modify addresses or shares in the future)
    - Definable onlySplitAfterLocked in deployment
- Useful check functions:
    - Check if split is equal between all beneficiaries
    - Check if address is already an beneficiary



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

## Tests

For this, you need to have foundry installed.

Run anvil for local blockchain:
```
anvil
```

Export dummy mnemonic for simplicity
```
export MNEMONIC="test test test test test test test test test test test junk"
```

Deploy splitter contract:
```
cast send --rpc-url localhost:8545 --mnemonic $MNEMONIC --mnemonic-index 0 --create $(cat output/Splitter/Splitter.bin) $(cast abi-encode "__init__(bool,uint256)" false 0)
```

If using default anvil parameters, contract should be deployed here (update if not)
```
export SPLITTER_CONTRACT=0x5FbDB2315678afecb367f032d93F642f64180aa3
```

We can add some addresses:
```
cast send $SPLITTER_CONTRACT "addAddress(address,uint256)" 0x0000000000000000000000000000000000000000 3 --mnemonic $MNEMONIC --mnemonic-index 0

cast send $SPLITTER_CONTRACT "addAddress(address,uint256)" 0x0000000000000000000000000000000000000001 2 --mnemonic $MNEMONIC --mnemonic-index 0
```

And check if the split is equal, which will not be the case:
```
cast call $SPLITTER_CONTRACT "checkIfEqualSplit()"
> 0x0000000000000000000000000000000000000000000000000000000000000000
```

We can update the shares of the second address to make it equal: 
```
cast send $SPLITTER_CONTRACT "updateShareForAddress(address,uint256)" 0x0000000000000000000000000000000000000001 3 --mnemonic $MNEMONIC --mnemonic-index 0
```

And check again:
```
cast call $SPLITTER_CONTRACT "checkIfEqualSplit()"
> 0x0000000000000000000000000000000000000000000000000000000000000001
```

Lets add a third address:
```
cast send $SPLITTER_CONTRACT "addAddress(address,uint256)" 0x0000000000000000000000000000000000000002 1 --mnemonic $MNEMONIC --mnemonic-index 0
```

Check the N of beneficiaries, should be 3:
```
cast call $SPLITTER_CONTRACT "getNumberBeneficiaries()"
> 0x0000000000000000000000000000000000000000000000000000000000000003
```

Lets send the contract some ETH and review the balance:
```
cast send $SPLITTER_CONTRACT --value 1ether --mnemonic $MNEMONIC --mnemonic-index 0

cast balance $SPLITTER_CONTRACT
> 1000000000000000000
```

Now, lets split the balance:
```
cast send $SPLITTER_CONTRACT "splitBalance()" --mnemonic $MNEMONIC --mnemonic-index 0
```

We can check the balances, it should be in a proportion of 3/3/1:
```
cast balance 0x0000000000000000000000000000000000000000
>433012991571428571
cast balance 0x0000000000000000000000000000000000000001
>428571428571428571
cast balance 0x0000000000000000000000000000000000000002
>142857142857142858
```

We can remove an address (the second one for example):
```
cast send $SPLITTER_CONTRACT "removeAddress(address)" 0x0000000000000000000000000000000000000001 --mnemonic $MNEMONIC --mnemonic-index 0
```

And check the N beneficiaries again:
```
cast call $SPLITTER_CONTRACT "getNumberBeneficiaries()"
> 0x0000000000000000000000000000000000000002
```






