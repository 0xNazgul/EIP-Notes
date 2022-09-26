# EIP-3607: Reject transactions from senders with deployed code
Currently addresses are only 160 bits long it is possible to create a collision between a contract account and an (EOA) using an estimated `2**80` computing operations, which is feasible now given a large budget. The fix is to never allow to use an address that already has code deployed as an EOA address.

## Motivation
By creating keys for `2**80` EOAs and simulating the deployment of `2**80` contracts from these EOAs (one each), one expects to find about one collision where an EOA has the same address as one contract.
This very simple form of the attack requires the storage of `2**80` addresses, which is a practical barrier: It would require `2.4*10**25` bytes of memory. However, there are cycle finding algorithms that can perform the collision search without requiring large amounts of storage. It's estimated that a collision between a contract and an EOA could be found in about one year with an investment of ca. US$10 billion in hardware and electricity.

## Specification
Any TX where `tx.sender` has a `CODEHASH != EMPTYCODEHASH` must be rejected as invalid (`EMPTYCODEHASH = 0xc5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470`). The invalid TX must be rejected by the client and not be included in a block and a block containing such a TX must be considered invalid.
