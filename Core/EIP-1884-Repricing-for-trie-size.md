# EIP-1884: Repricing for trie-size-dependent opcodes
Growth of the Ethereum state has caused certain opcodes to be more resource-intensive so a raise of `gasCost` is due. 

## Motivation
Imbalance between the price of an operation and resource consumption has several drawbacks:
* Could be used for an attack, by filling blocks with underpriced operations which causes excessive block processing time.
* Underpriced opcodes cause a skewed block gas limit where sometimes blocks finish quickly but other blocks with similar gas use finish slowly.

## Specification
At block `N`:
* The `SLOAD 0x54` operation changes from 200 to 800 gas,
* The `BALANCE 0x31` operation changes from 400 to 700 gas,
* The `EXTCODEHASH 0x3F` operation changes from 400 to 700 gas,
* A new opcode, `SELFBALANCE 0x47`:
	* `SELFBALANCE` pops 0 arguments off the stack,
	* `SELFBALANCE` pushes the `balance` of the current address to the stack,
	* `SELFBALANCE` is priced as `GasFastStep`, at 5 gas.

## Rationale
lot of cool [graphs](https://eips.ethereum.org/EIPS/eip-1884#rationale) here to look at 