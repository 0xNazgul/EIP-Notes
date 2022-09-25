# EIP-2929: Gas cost increases for state access opcodes
Increases gas cost for `SLOAD, *CALL, BALANCE, EXT* and SELFDESTRUCT` when used for the first time in a TX.
* `SLOAD 0x54` to 2_100
* `*CALL` opcode family (`0xf1, f2, f4, fA`), `BALANCE 0x31` and the `EXT*` opcode family (`0x3b, 0x3c, 0x3f`) to 2_600
* Exempts precompiles and addresses and storage slots that have already been accessed in the same TX. 
* Reforms `SSTORE` metering and `SELFDESTRUCT` to ensure de-facto storage loads inherent in those opcodes are priced correctly.

## Motivation
The main function of gas costs is to be an estimate of the time needed to process the opcode with the goal being for the gas limit to correspond to a limit on the time needed to process a block. Storage-accessing opcodes have been underpriced. Benefits:
1. Improvements in database layout that involve redesigning the client to read storage directly.
2. Performs most of the work needed to make [stateless witness sizes](https://ethereum-magicians.org/t/protocol-changes-to-bound-witness-size/3885) in Ethereum acceptable. [A swithc to binary tries](https://ethresear.ch/t/binary-trie-format/7621) the theoretical maximum witness size not including code size would decrease from `(12500000 gas limit) / (700 gas per BALANCE) * (800 witness bytes per BALANCE) ~= 14.3M bytes to 12500000 / 2600 * 800 ~= 3.85M bytes`
3. `SNARK/STARK` witnesses will benifit since they will be able to prove 10_000 Rescue hashes per second (consumer desktop). Assuming 25 hashes per Merkle branch and block full of state accesses.
	* Currently: `12500000 / 700 * 25 / 10000 ~= 44.64` seconds to generate
	* After: `12500000 / 2500 * 25 / 10000 ~= 12.5` seconds

## Specification
* FORK_BLOCK: 12_244_000
* COLD_SLOAD_COST: 2_100
* COLD_ACCOUNT_ACCESS_COST:	2_600
* WARM_STORAGE_READ_COST: 100

When a transaction execution begins,
* `accessed_storage_keys` is initialized to empty, and
* `accessed_addresses` is initialized to include
	* the `tx.sender, tx.to` 
	* and the set of all precompiles.

## Storage read changes
When an address is either the target of a `EXTCODESIZE 0x3B, EXTCODECOPY 0x3C, EXTCODEHASH 0x3F or BALANCE 0x31` opcode or the target of a `CALL 0xF1, CALLCODE 0xF2, DELEGATECALL 0xF4, STATICCALL 0xFA` opcode, the gas costs are computed as follows:
* If the target is not in `accessed_addresses`, charge `COLD_ACCOUNT_ACCESS_COST` gas, and add the address to `accessed_addresses`.
* Otherwise, charge `WARM_STORAGE_READ_COST` gas.

In all cases, the gas cost is charged and the map is updated at the time that the opcode is being called. When a `CREATE or CREATE2` opcode is called, immediately add the address being created to `accessed_addresses`, but gas costs of `CREATE and CREATE2` are unchanged. 
* Clarification: If a `CREATE/CREATE2` operation fails later on (e.g during the execution of `initcode` or has insufficient gas to store the code in the state) the address of the contract itself remains in `access_addresses` (but any additions made within the inner scope are reverted).

For `SLOAD`, if the (`address, storage_key`) pair (where `address` is the address of the contract whose storage is being read) is not yet in `accessed_storage_keys`, charge `COLD_SLOAD_COST` gas and add the pair to `accessed_storage_keys`. If the pair is already in `accessed_storage_keys`, charge `WARM_STORAGE_READ_COST` gas.
* For call-variants, the `100/2_600` cost is applied immediately (exactly like how 700 was charged before this EIP), i.e: before calculating the `63/64ths` available for entering the call.
* There is currently no way to perform a cold sload read/write on a cold account, simply because in order to read/write a slot, the execution must already be inside the account. The behaviour of cold storage reads/writes on cold accounts is undefined. Any future EIP which proposes to add remote read/write would need to define the pricing behaviour of that change.

## SSTORE changes
When calling `SSTORE`, check if the (`address, storage_key`) pair is in `accessed_storage_keys`. If it is not, charge an additional `COLD_SLOAD_COST` gas, and add the pair to `accessed_storage_keys`. 

Modify the parameters defined in `EIP-2200` as follows:

Parameter: Old value -> New value
* `SLOAD_GAS`: 800 -> `= WARM_STORAGE_READ_COST`
* `SSTORE_RESET_GAS`: 5_000 -> `5000 - COLD_SLOAD_COST`

The constant `SLOAD_GAS` is used in several places in EIP-2200, e.g `SSTORE_SET_GAS - SLOAD_GAS`.

## `SELFDESTRUCT` changes
If the ETH recipient of a `SELFDESTRUCT` is not in `accessed_addresses` (regardless of whether or not the amount sent is nonzero), charge an additional `COLD_ACCOUNT_ACCESS_COST` on top of the existing gas costs, and add the ETH recipient to the set.
* `SELFDESTRUCT` does not charge a `WARM_STORAGE_READ_COST` in case the recipient is already warm, this differs from how the other call-variants work. This is to keep the changes small, a `SELFDESTRUCT` already costs 5_000 and is a no-op if invoked more than once.