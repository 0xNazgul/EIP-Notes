# EIP-2976 Typed Transactions over Gossip
Typed TXs can be snt over `devp2p` as `TransactionType || TransactionPayload`. The contents of `TransactionPayload` are defined by `TransactionType` and clients may start supporting their gossip without incrementing the `devp2p` version. 
* If a client receives a `TransactionType` that it doesn't recognize, it should disconnect from the peer who sent it. Clients must not send new TX types before they believe the fork block is reached.

## Specification
Definitions:
* `||` : The byte/byte-array concatenation operator.
* `|`: The type union operator.
* `DEVP2P_VERSION = TBD`
* `Transaction`: Either `TypedTransaction or LegacyTransaction`
* `TypedTransaction`: A byte array containing `TransactionType || TransactionPayload`
* `TypedTransactionHash`: `keccak256(TypedTransaction)`
* `TransactionType`: A positive unsigned 8-bit number between `0 and 0x7f` that represents the type of the TX
* `TransactionPayload`: An opaque byte array whose interpretation is dependent on the `TransactionType`
* `LegacyTransaction`: An array of the form `[nonce, gasPrice, gasLimit, to, value, data, v, r, s]`
* `LegacyTransactionHash`: `keccak256(rlp(LegacyTransaction))`
* `TransactionId`: `keccak256(TypedTransactionHash | LegacyTransactionHash)`
* `Receipt`: Either `TypedReceipt or LegacyReceipt`
* `TypedReceipt`: A byte array containing `TransactionType || ReceiptPayload`
* `ReceiptPayload`: An opaque byte array whose interpretation is dependent on the `TransactionType`
* `LegacyReceipt`: An array of the form `[status, cumulativeGasUsed, logsBloom, logs]`
* `LegacyReceiptHash`: `keccak256(rlp(LegacyReceipt))`

### Protocol Behavior
* If a client receives a `TransactionType` it doesn't recognize via any message, it should disconnect the peer that sent it
* If a client receives a `TransactionPayload` that isn't valid for the `TransactionType` it should disconnect the peer that sent it
* Clients must not send TXs of a new `TransactionType` until that TX type's introductory fork block.
* Clients may disconnect peers who send TXs of a new `TransactionType` significantly before that TX type's introductory fork block

### Protocol Messages
* `Transactions 0x02`: `[Transaction_0, Transaction_1, ..., Transaction_n]`
* `BlockBodies 0x06`: `[BlockBody_0, BlockBody_1, ..., BlockBody_n]` where:
	* `BlockBody` is `[TransactionList, UncleList]`
	* `TransactionList` is `[Transaction_0, Transaction_1, ..., Transaction_n]`
	* `UnclesList` is defined in previous versions of the `devp2p` specification
* `NewBlock 0x07: `[[BlockHeader, TransactionList, UncleList], TotalDifficulty]` where:
	* `BlockHeader` is defined in previous versions of the `devp2` specification
	* `TransactionList is `[Transaction_0, Transaction_1, ..., Transaction_n]`
	* `UnclesList` is defined in previous versions of the `devp2p` specification
	* `TotalDifficulty` is defined in previous versions of the `devp2p` specification
* `NewPooledTransactionIds 0x08`: `[TransactionId_0, TransactionId_1, ..., TransactionId_n]`
* `GetPooledTransactions 0x09`: `[TransactionId_0, TransactionId_1, ..., TransactionId_n]`
* `PooledTransactions 0x0a`: `[Transaction_0, Transaction_1, ..., Transaction_n]`
* `Receipts 0x10`: `[ReceiptList_0, ReceiptList_1, ..., ReceiptList_n]` where:
	* `ReceiptList` is `[Receipt_0, Receipt_1, ..., Receipt_n]`