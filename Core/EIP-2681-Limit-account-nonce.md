# EIP-2681: Limit account nonce to 2^64-1
Limit account nonce to be between `0 and 2^64-1`

## Motivation
Currently they are specified to be arbitrarily long uint. Dealing with this length data in the state is not optimal and so this will allow proofs to represent the nonce in more optimized way.
* Also proves beneficial to TX formats
* Facilitates minor optimization in clients for the nonce no longer need to be in a 256-bit number

## Specification
Two new restrictions retroactively from genesis:
1. Any TX is invalid, where the nonce `>= 2^64-1`.
2. `CREATE and CREATE2` instructions execution ends with the result 0 pushed on stack, where the account nonce is `2^64-1`. Gas for `initcode` execution is not deducted in this case.


