# EIP-1052 EXTCODEHASH
This new opcode will return the `keccak256` hash of a contract’s code. Some contracts need to perform checks on a contracts bytecode but don't need the bytecode itself. 
* Currently contracts use `EXTCODECOPY` but this is expensive in cases where only the hash is required.

## Specification
`EXTCODECOPY 0x3f` will take one argument from the stack, zeros the first 96 bits and pushes to the stack the `keccak256` hash of the code of the account at the address being the remaining 160 bits.
* If account is empty/non-exist 0 is pushed to the stack.
* If account doesn't have code the `keccak256` hash of empty data is pushed to the stack.
* Gas cost `EXTCODECOPY` is `400`

## Rationale
Will save gas for man cases and be widely useful. Gas cost is the same as `BALANCE` because the execution of the `EXTCODEHASH` requires the same account lookup.
* Only the 20 last bytes of the argument are significant (first 12 bytes is ignored) and has the same semantics of the `BALANCE, EXTCODESIZE AND EXTCODECOPY`.