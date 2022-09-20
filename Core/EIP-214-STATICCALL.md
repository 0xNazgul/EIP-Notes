# EIP-214 STATICCALL
To increase smart contract security, `STATICCALL` can be used to call another contract (or itself) without doing any modifications to the state during the call (and its subcalls, if present). If any opcode attempts to perform such modification it will result in an exception.

## Specification
Introdouces a new `STATIC` flag that is default `false`. It's value is always copied to sub-calls with an exception for `STATICCALL`
* `STATICCALL 0xfa`

`STATICCALL` is equivalent to `CALL` but it takes only 6 arguments (no value argument) and calls the child with the `STATIC` flag set to `true` for the execution of the child. 
* At the end of the call `STATIC` flag is reset.
* Operations that will throw an exception when `STATIC` is set `true` are `CREATE, CREATE2, LOG0, LOG1, LOG2, LOG3, LOG4, SSTORE, and SELFDESTRUCT`
* `CALLCODE` is considered not state-changing even with a non-zero value.


