# EIP-161: State trie clearing (invariant-preserving alternative) 

## Specification
1. Account creation TXs and `CREATE` operation `SHALL`, prior to the execution of initialization code, increment the nonce over and above its normal starting value by 1
2. Whereas `CALL` and `SUICIDE` would charge `25_000` gas when the destination is non-existent, now the charge `SHALL` only be levied if the operation transfers more than 0 value and the destination account is dead.
3. No account may change state from non-existent to existent-but-empty. If an operation would do this the account `SHALL` instead remain non-existent.
4. At the end of the TX any account touched by the execution of that transaction which is now empty `SHALL` instead become non-existent.

Where:
* An account is considered to be touched when it is involved in any potentially state-changing operation (including being a recipient of a 0 value transfer).
* An account is considered empty when it has no code, 0 nonce and 0 balance.
* An account is considered dead when either it is non-existent or it is empty.
* At the end of the TX is immediately following the execution of the suicide list, prior to the determination of the state trie root for receipt population.

An account changes state when:
* Is the target or refund of a `SUICIDE` or 0 or more value
* Is the source or destination of a `CALL` or message-call TX transferring 0 or more value
* IS the source or creation of a `CREATE` or contract-creation Tx endowing 0 or more value
* The block author it is the recipient of block-rewards or TX fees of 0 or more value

