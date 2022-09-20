# EIP-211 New opcodes RETURNDATASIZE and RETURNDATACOPY
These are mechanisms for returning arbitrary-length data.

## Motivation
Allows for a function to be able to return data whose length cannot be anticipated before the call. This was done by splitting the call into two calls, with the first to compute the size and this was expensive. 
* Compiler implementors are to reserve a zero-length area for return data. If the size of the return data is unknown before the call? Then use `RETURNDATACOPY` in conjunction with `RETURNDATASIZE` to actually retrieve the data.

## Specification
If `block.number >= BYZANTIUM_FORK_BLKNUM` two new opcodes can be used and are amended to the semantics of any opcode that creates a new call frame. EVM call frame has a new internal buffer of variable size called the return data buffer. 
* It is created empty for each new call frame. on executing any call-like opcode the buffer is cleared.
* After executing call-like opcode the return dat of the call is stored in the return data buffer (of the caller). 
* Exception, `CREATE` and `CREATE2` are considered to return the empty buffer in the success case and the failure data in the failure case.
* If it does not really instantiate a call from the return data buffer is empty.

`RETURNDATASIZE: 0x3d`
* Pushes the size of the return data buffer onto the stack. Costs 2.

`RETURNDATACOPY: 0x3e`
* Similar semantics to `CALLDATACOPY` BUT instead of copying data from the call data, it does so from the return data buffer.
* Accessing the return data buffer beyond it's size results in a failure.
* Cost `3 + 3 * ceil(amount / 32)`