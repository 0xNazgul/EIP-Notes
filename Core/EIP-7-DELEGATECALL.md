# EIP-7: DELEGATECALL

Adds a new `DELEGATECALL 0xf4` as new opcode. This is similar `CALLCODE` but it propagates the sender and value from the parent scope to the child scope.

##  Specification
`DELEGATECALL 0xf4`:
1. gas: amount of gas the code can use to execute
2. to: destination address that code is executed
3. in_offset: offset into memory of the input
4. in_size: size of input (bytes)
5. out_offset: offset inot memory of the output
6. out_size: the zie of the scratch pad for the output

NOTES:
1. Gas
* Is the total amount the callee receives
* Upfront gas cost is always `schedule.callGas + gas` like `CALLCODE` (account creation never happens)
* Unused gas is refunded

2. Sender
* `CALLER && VALUE` act the same in the callee's environment as they do in the caller's environment
* Depth limit is still 1024

## Rationale
`DELEGATECALL` makes it easier for a contract to store another address as mutable source of code and pass through calls. The child code would execute in essentially the same environment
* Except for reduced gas and increased callstack depth
