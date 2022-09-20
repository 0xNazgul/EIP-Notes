# EIP-1014 Skinny CREATE2

## Specification
New opcode `CREATE2 0Xf5` takes 4 stack arguments: `endowment, memory_start, memory_length, salt`. 
* Behaves just like `CREATE` but uses `keccak256( 0xff ++ address ++ salt ++ keccak256(init_code))[12:]` instead of the usual sender-and-nonce-hash as the address where the contract is initialized at.

* `CREATE2` has the same gas schema as `CREATE` butt an extra `hashcost: GSHA3WORD * ceil(len(init_code) / 32)`, to account for the hashing. This is deducted at the same time as memory-expansion gas.
* `CreateGas` is deducted before evaluation of the resulting address and execution of `init_code`.
	* `0xff` is a single byte,
	* `address` is always 20 bytes,
	* `salt` is always 32 bytes (a stack item).
The preimage for the final hashing round is exactly 85 bytes long.

## Motivation
Allows interactions to be made with addresses that do not exist yet on-chain but can be relied on to eventually contain code that has been created by a particular piece of init code. 

## Rationale
Address formula:
* Ensures that addresses created with this scheme cannot collide with addresses created using the traditional `keccak256(rlp([sender, nonce]))` formula, as `0xff` can only be a starting byte for RLP for data many petabytes long.
* Ensures that the hash preimage has a fixed size.

## Gas cost
Since address calculation depends on hashing the `init_code`, it would leave clients open to DoS attacks if executions could repeatedly cause hashing of large pieces of `init_code`, since expansion of memory is paid for only once. This will use the same cost-per-word as the `SHA3` opcode.

## Clarifications
The `init_code` is the code that, when executed, produces the runtime bytecode that will be placed into the state and which typically is used by high level languages to implement a ‘constructor’.
This EIP makes collisions possible. The behaviour at collisions is specified by [EIP-684](https://github.com/ethereum/EIPs/issues/684)
* Specifically, if `nonce or code` is nonzero, then the create-operation fails.
With [EIP-161](https://eips.ethereum.org/EIPS/eip-161)
* This means that if a contract is created in a transaction, the `nonce` is immediately non-zero, with the side-effect that a collision within the same transaction will always fail (even if it’s carried out from the `init_code` itself).
* It should also be noted that `SELFDESTRUCT 0xff` has no immediate effect on `nonce or code`, so a contract cannot be destroyed and recreated within one transaction.
