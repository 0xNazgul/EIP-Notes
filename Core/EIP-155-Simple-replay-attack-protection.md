# EIP-155: Simple replay attack protection

## Specification
If `block.number >= FORK_BLKNUM` and `CHAIN_ID`, When computing the hash of a transaction for the purposes of signing, instead of hashing only six rlp encoded elements (`nounce, gasprice, startgas, to, value, data`) you should hash nine (`nonce, gasprice, startgas, to, value, data, chainid, 0, 0`).

If you do then the `v` of the signature must be `{0, 1} + CHAIN_ID * 2 + 35` where `{0, 1}` is the parity of the `y` value of the curve point for which `r` is the x-value in the secp256k1 signing process.
* If you choose to only hash six values then `v` continues to be set to `{0, 1} + 27`.

If `block.number >= FORK_BLKNUM` and `v = CHAIN_ID * 2 + 35` or `v = CHAIN_ID * 2 + 36`, when computing the hash of a TX for recovering, instead of hashing only six rlp encoded elements (`nounce, gasprice, startgas, to, value, data`) you should hash nine (`nonce, gasprice, startgas, to, value, data, chainid, 0, 0`).
* The current existing signature scheme using `v = 27` and `v = 28` remains valid and still operates under the same rules.

# Rationale
This provides a way to send transaction that would work on Ethereum but it won't work on another test chain.
