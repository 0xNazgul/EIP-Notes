# EIP-145: Bitwise shifting instructions in EVM
Before there was no native bitwise shifting instruction that was efficient enough to be on par with other arithmetic operations.

# Specification
`0x1b SHL` shift left:
* Pops two values from the stack, first `arg1` and then `arg2`
* Pushes on the stack `arg2` shifted to the left by `arg1` number of bits
Equal to:
`(arg2 * 2^arg1) mod 2^256`

* Both `arg` values are interpreted as an unsigned number
* If `arg1 >= 256` it results 0
* Equivalent to `PUSH1 2 EXP MUL`

`0x1c SHR` logical shift right:
* Pops two values from the stack, first `arg1` and then `arg2`
* Pushes on the stack `arg2` shifted to the right by `arg1` number of bits with zero fill
Equal to:
`floor(arg2 / 2^arg1)`

* Both `arg` values are interpreted as an unsigned number
* If `arg1 >= 256` it results 0
* Equivalent to `PUSH1 2 EXP DIV`

`0x1d SAR` arithmetic shift right:
* Pops two values from the stack, first `arg1` and then `arg2`
* Pushes on the stack `arg2` shifted to the right by `arg1` number of bits with sign extension

Equal to:
`floor(arg2 / 2^arg1)`

* `arg2` is interpreted as a signed number
* `arg1` is interpreted as an unsigned number
* If `arg1 >= 256` the result is 0 if `arg2` is non-negative or -1 if `arg2` is negative
* NOT EQUAL to `PUSH1 2 EXP SDIV` because of rounding. 

All instructions cost 3 gas.