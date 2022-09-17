# EIP-152: Add BLAKE2 compression function `F` precompile
Introduces a new precompiled contract that has a compression function `F` used in the BLAKE2 cryptographic hashing algorithm to allow interoperability between the EVM and Zcash. Also a more flexible cryptographic hash primitives to the EVM.

## Motivation
BLAKE2 allows for efficient verification of Equilhash PoW used in Zcash making a BTC Relay (style SPV client) on Ethereum. A single verification of an Equilhash PoW verification requires 512 iterations of the hash function. This makes verification of Zcash block headers expensive if BLACKE2 is used.

BLAKE2b is a 64-bit BLAKE2 variant taht is highly optimized and faster than MD5. This will enable contracts to do trustless atomic swaps between the chains providing a much needed aspect of privacy.

## Specification
The precompiled contract is at `0x09` wrapping the [BLAKE2](https://www.rfc-editor.org/rfc/rfc7693#section-3.2). The precompile requires 6 inputs encoded, taking exactly 213 bytes:
* rounds - the number of rounds -32-bit unsigned big-endian word
* h - the stae vector - 8 unsigned 64-bit little-endian words
* m - the message block vector - 16 unsigned 64-bit little-endian words
* t_0, t_1 - offset counters - 2 unsigned 64-bit little-endian words
* f - the final block indicator flag - 8-bit word

`[4 bytes for rounds][64 bytes for h][128 bytes for m][8 bytes for t_0][8 bytes for t_1][1 byte for f]`

`f` parameter is `true` if set to 1 and `false` if set to 0. All other values are invalid encoding. `F` function computed as [specified](https://www.rfc-editor.org/rfc/rfc7693#section-3.2) and return the updated state vector `h` with unchanged encoding (little-endian).

## Gas cost
Each operation will cost `GFROUND * rounds` gas where `GFROUND = 1`
