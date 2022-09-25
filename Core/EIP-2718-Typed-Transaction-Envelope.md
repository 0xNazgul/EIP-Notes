# EIP-2718: Typed Transaction Envelope
`TransactionType || TransactionPayload` is a valid TX and `TransactionType || ReceiptPayload` is a valid TX receipt where `TransactionType` identifies the format of the TX and `*Payload` is the TX/receipt contents, which are defined in future EIPs.

## Specification
`||` is the byte/byte-array concatenation operator.
1. TXs
TX root in the block header `MUST` be the root hash of `patriciaTrie(rlp(Index) => Transaction)` where:
* `Index` is the index in the block of this transaction
* `Transaction` is either `TransactionType || TransactionPayload or LegacyTransaction`
* `TransactionType` is a positive uint 8-bit number between `0 and 0x7f` that represents the type of the TX
* `TransactionPayload` is an opaque byte array whose interpretation is dependent on the `TransactionType` and defined in future EIPs
* `LegacyTransaction` is `rlp([nonce, gasPrice, gasLimit, to, value, data, v, r, s])`

All signatures for future TX types `SHOULD` include the `TransactionType` as the first byte of the signed data. This makes it so we do not have to worry about signatures for one TX type being used as signatures for a different TX type.

2. Receipts
Receipt root in the block header `MUST` be the root hash of `patriciaTrie(rlp(Index) => Receipt)` where:
* `Index` is the index in the block of the TX this receipt is for
* `Receipt` is either `TransactionType || ReceiptPayload or LegacyReceipt`
* `TransactionType` is a positive uint 8-bit number between `0 and 0x7f` that represents the type of the TX
* `ReceiptPayload` is an opaque byte array whose interpretation is dependent on the `TransactionType` and defined in future EIPs
* `LegacyReceipt` is `rlp([status, cumulativeGasUsed, logsBloom, logs])`

The `TransactionType` of the receipt `MUST` match the `TransactionType` of the TX with a matching `Index`.

## Security Considerations
When designing a new 2718 TX type, it's `STRONGLY` recommended to include the TX type as the first byte of the signed payload. If you fail to do this, it is possible that your TX may be signature compatible with TXs of another type which can introduce vulnerabilities for users.