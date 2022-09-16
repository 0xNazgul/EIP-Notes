# EIP-140 REVERT instruction
`REVERT` will provide a way to stop execution, revert state changes, only consume gas used (not all gas provided) and can return a reason.

## Motivation
There was only two ways to revert a TX: out-of-gas and invalid instruction, both of them consume all gas. These also revert all changes and `LOGS` with no reason of aborting.

## Specification
`REVERT` opcode is `0xfd` that takes two stack items: memory_offset and memory_length (or error message) it will:
1. Abort execution
2. Roll back state changes
3. Return unused gas 
4. Return a message

Cost is equal to the `RETURN` instruction and the contract only has to pay for memory. If no gas is left to cover the cost there is a stack underflow and results in a out-of-gas and returns 0 on the stack following `REVERT`.