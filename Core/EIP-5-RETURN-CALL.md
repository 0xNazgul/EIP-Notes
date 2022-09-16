# EIP-5: Gas Usage for `RETURN` and `CALL`

## Specification
Size of the output has to be specified in advanced when calling another contract/functions taht return strings/dynamically-sized arrays. Because of this gas has to be paid for memory that is not written to. Making it costly/inflexible pretty much unusable.

Gas and memory semantics for `CALL` and `DELEGATECALL` is changed by (NOTE: create is not changed because it doesn't write to memory):
	* EX: `CALL` arguments of `gas, address, value, input_start, input_size, output_start, output_size` the gas for growing memory is charged for `input_start + input_size` NOT `output_start + output_size`.
	* The called contract returns data of size `n`, the memory of the calling contract (The payer of the gas) is `output_start + min(output_size, n)` and output is written to `[output_start, output_start + min(n, output_size))`. 
	* This allows the calling contract to run out of gas at the start and end of the opcode. 
	* `MSIZE` Should return the size the memory grown to

## Motivation
It's good practise to reserve a memory area for output of a call instead of a subroutine to write to arbitrary areas in memory. On top of this knowing the output size prior to a call is hard and if the data is in storage this is even harder. The other part is charging gas for areas of memory not actually written to is unnecessary.

To solve this the caller can provide the area of memory at the end of their memory area. The callee can write to by returning and the caller is only charged for the memory area that is written. So, the area of memory that is actually written to is `(output_start, MSIZE)` (`MSIZE` evaluated after the call). It will allow `proxy` contracts that call other contracts with a interface they don't know and return only the output.
