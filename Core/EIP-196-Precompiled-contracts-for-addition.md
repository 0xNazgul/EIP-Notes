# EIP-196: Precompiled contracts for addition and scalar multiplication on the elliptic curve alt_bn128
Precompiled contracts for elliptic curve operations are required in order to perform `zkSNARK` verification within the block gas limit.

## Motivation
Current contract executions are transparent making them unsuitable for several use-cases that involve private information like the location, identity or history of pas TXs. `zkSNARK`s can solve this but it is currently too expensive to fit the block gas limit.

## Specification
If `block.number >= BYZANTIUM_FORK_BLKNUM`, add precompiled contracts for point addition (`ADD`) and scalar multiplication (`MUL`) on the elliptic curve `alt_bn128.
* `ADD` address `0x6`
* `MUL` address `0x7`

Curve is as such:
```js
Y^2 = X^3 + 3
over the field F_p with
p = 21888242871839275222246405745257275088696311157297823662689037894645226208583
```

## Encoding
* Field elements and scalars are encoded as 32 byte big-endian numbers. Curve points are encoded as two field elements `(x, y)` where the point at infinity is encoded as `(0, 0)`.
* Tuples of objects are encoded as their concatenation
* For both precompiled contracts, if the input is shorter than expected it is assumed to be virtually padded with 0s at the end. If too long extra bytes at the end are ignored.
* Length of returned data is always as specified.

## Exact semantics
Invalid input: if any input point doesn't lie on the curve or any of the field elements is equal or larger than the field modulus p, the contract fails.
* The scalar can be any number between 0 and `2**256-1`

`ADD`
* Input: two curve points `(x, y)`.
* Output: curve point `x + y` where `+` is point addition on the elliptic curve `alt_bn128` specified above.
* Fails on invalid input and consumes all gas provided.

`MUL` 
* Input: curve point and scalar `(x, s)`.
* Output: curve point `s 8 x` where `*` is the scalar multiplication on the elliptic curve `alt_bn128`.
* Fails on invalid input and consumes all gas.

## Gas costs
* Gas cost for `ECADD`: `500`
* Gas cost for `ECMUL`: `40_000`

## Rationale
`alt_bn128` was picked for it's verification building block of pairing function. It also has synergy effects with ZCash and re-use some of their components and artifacts.
