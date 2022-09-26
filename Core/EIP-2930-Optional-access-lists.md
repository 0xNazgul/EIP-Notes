# EIP-2930 Optional access lists
Adds a transaction type which contains an access list a list of addresses and storage keys that the TX plans to access. 
* Outside the list accesses is possible but more expensive
* EIP-2718 introduced a new TX type, `accessList`, this specifies a list of addresses and storage keys.
	* These are added into `accessed_addresses and accessed_storage_keys` global sets
	* Gas cost is charged though at a discount relative to the cost of accessing outside the list

## Motivation
1. Mitigates contract breakage risks introduced by EIP-2929, as TXs could pre-specify and pre-pay for the accounts and storage slots that the TX plans to access; as a result, in the actual execution, the `SLOAD and EXT*` opcodes would only cost 100 gas. Low enough that it would not only prevent breakage due to that EIP but also unstuck any contracts that became stuck due to EIP 1884.
2. The access list format and the logic for handling the format. This logic can later be repurposed for many other purposes, including block-wide witnesses, use in ReGenesis, moving toward static state access over time, and more.

## Specification
Definitions & parameters:
`TransactionType`: 1 (See EIP-2718).
`ChainId`: The TX only valid on networks with this `chainID`.
`YParity`: The parity (0 for even, 1 for odd) of the y-value of a `secp256k1` signature.
`FORK_BLOCK`:12244000
`ACCESS_LIST_STORAGE_KEY_COST`:1900
`ACCESS_LIST_ADDRESS_COST`:2400

* As of `FORK_BLOCK_NUMBER`, a new TX is introduced with `TransactionType 1`. 
* The `TransactionPayload` for this TX is `rlp([chainId, nonce, gasPrice, gasLimit, to, value, data, accessList, signatureYParity, signatureR, signatureS])`. 
* The `signatureYParity, signatureR, signatureS` elements of this TX represent a `secp256k1` signature over `keccak256(0x01 || rlp([chainId, nonce, gasPrice, gasLimit, to, value, data, accessList]))`. 
* The `ReceiptPayload` for this TX is `rlp([status, cumulativeGasUsed, logsBloom, logs])`.
* For the TX to be valid, `accessList` must be of type `[[{20 bytes}, [{32 bytes}...]]...]` (where `...` means zero or more of the thing to the left). 

At the beginning of execution we charge additional gas for the access list: `ACCESS_LIST_ADDRESS_COST` gas per address and `ACCESS_LIST_STORAGE_KEY_COST` gas per storage key. 
* Non-unique addresses and storage keys are not disallowed, though they will be charged for multiple times, and aside from the higher cost.

The address and storage keys would be immediately loaded into the `accessed_addresses and accessed_storage_keys` global sets. 
* This can be done using the following logic (which doubles as a specification-in-code of validation of the RLP-decoded access list):

```
def process_access_list(access_list) -> Tuple[List[Set[Address], Set[Pair[Address, Bytes32]]], int]:
    accessed_addresses = set()
    accessed_storage_keys = set()
    gas_cost = 0
    assert isinstance(access_list, list)
    for item in access_list:
        assert isinstance(item, list) and len(item) == 2
        # Validate and add the address
        address = item[0]
        assert isinstance(address, bytes) and len(address) == 20
        accessed_addresses.add(address)
        gas_cost += ACCESS_LIST_ADDRESS_COST
        # Validate and add the storage keys
        assert isinstance(item[1], list)
        for key in item[1]:
            assert isinstance(key, bytes) and len(key) == 32
            accessed_storage_keys.add((address, key))
            gas_cost += ACCESS_LIST_STORAGE_KEY_COST
    return (
        accessed_addresses,
        accessed_storage_keys,
        gas_cost
    )
```
The access list is `NOT` charged per-byte fees like TX data is. The per-item costs described above are meant to cover the bandwidth costs of the access list data in addition to the costs of accessing those accounts and storage keys when evaluating the TX.
