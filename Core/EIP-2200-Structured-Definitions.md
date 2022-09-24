## EIP-2200: Structured Definitions for Net Gas Metering
Will implement net gas metering with a structured definition so as to make it interoperable with other gas changes. This will be done for `SSTORE` opcode enabling new usages for contract storage and reduce gas cost.

## Motivation

Usages that benefits from this EIP’s gas reduction scheme includes:
* Subsequent storage write operations within the same call frame. This includes reentry locks, same-contract multi-send, etc.
* Exchange storage information between sub call frame and parent call frame, where this information does not need to be persistent outside of a transaction. This includes sub-frame error codes and message passing, etc.
`EIP-1283` introduced a new kind of reentrancy attacks with Solidity by default grants a stipend of 2_300 gas to simple transfer calls. This can be mitigated if `SSTORE` is not allowed in low gasleft state without breaking the backward compatibility.

## Specification
Define variables `SLOAD_GAS, SSTORE_SET_GAS, SSTORE_RESET_GAS and SSTORE_CLEARS_SCHEDULE`. The old and new values for those variables are:

* `SLOAD_GAS`: changed from 200 to 800.
* `SSTORE_SET_GAS`: 20_000, not changed.
* `SSTORE_RESET_GAS`: 5_000, not changed.
* `SSTORE_CLEARS_SCHEDULE`: 15_000, not changed.

Replace `SSTORE` opcode gas cost calculation (including refunds) with the following logic:
* If gasleft is less than or equal to gas stipend, fail the current call frame with ‘out of gas’ exception.
* If current value equals new value (this is a no-op), `SLOAD_GAS` is deducted.
* If current value does not equal new value
	* If original value equals current value (this storage slot has not been changed by the current execution context)
		* If original value is 0, `SSTORE_SET_GAS` is deducted.
		* Otherwise, `SSTORE_RESET_GAS` gas is deducted. If new value is 0, add `SSTORE_CLEARS_SCHEDULE` gas to refund counter.
	* If original value does not equal current value (this storage slot is dirty), `SLOAD_GAS` gas is deducted. Apply both of the following clauses.
		* If original value is not 0
			* If current value is 0 (also means that new value is not 0), remove `SSTORE_CLEARS_SCHEDULE` gas from refund counter.
			* If new value is 0 (also means that current value is not 0), add `SSTORE_CLEARS_SCHEDULE` gas to refund counter.
		* If original value equals new value (this storage slot is reset)
			* If original value is 0, add `SSTORE_SET_GAS - SLOAD_GAS` to refund counter.
			* Otherwise, add `SSTORE_RESET_GAS - SLOAD_GAS` gas to refund counter.

