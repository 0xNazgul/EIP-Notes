# EIP-197: Precompiled contracts for optimal ate pairing check on the elliptic curve alt_bn128
Precompiled contracts for elliptic curve pairing operations are required in order to perform `zkSNARK` verification within the block gas limit (Same abstract as EIP-196).

## Motivation
(Same info as EIP-196)

Pairing function can be used to perform a limited form of [multiplicatively homomorphic operations](https://en.wikipedia.org/wiki/Homomorphic_encryption). This can be used to run such computation within the block gas limit. 
* It only has a certain check and not an evaluation of a pairing function
* This was chosen for the reason that codomain of a pairing function is a rather complex field which could provide encoding problems.

## Specification
For `blocks where block.number >= BYZANTIUM_FORK_BLKNUM`, a precompiled contract for a bilinear function on groups on the elliptic curve `alt_bn128`. It will be defined in terms of a discrete logarithm. 
* The discrete logarithm is assumed to be hard to compute but an equivalent spec that makes use of elliptic curve pairing functions to be efficiently computed.
* Address `0x8`

For a cyclic group `G` (written additively) of prime order `q` let `log_P: G -> F_q` be the discrete logarithm on this group with respect to a generator `P`, (`log_P(x)`) is the smallest non-negative integer `n` such that `n * P = x`.

The contract is as follows, where the two groups `G_1` and `G_2` defined by their generators `P_1` and `P_2` below. Both generators have the same prime order `q`.
```js
Input: (a1, b1, a2, b2, ..., ak, bk) from (G_1 x G_2)^k
Output: If the length of the input is incorrect or any of the inputs are not elements of
        the respective group or are not encoded correctly, the call fails.
        Otherwise, return one if
        log_P1(a1) * log_P2(b1) + ... + log_P1(ak) * log_P2(bk) = 0
        (in F_q) and zero else.
```
* `K` is determined from the length of the input. `k` is the length of the input divided by `192`. If not a multiple of `192`, it fails. Empty input is valid and returns 1.

To check an that an input is an element of `G_1`, verifying the encoding of the coordinates and checking that they satisfy the curve equation is sufficient.`G_2`, the order of the element has to be checked to be equal to the group order `q = 21888242871839275222246405745257275088548364400416034343698204186575808495617`.

## Definition of the groups
* The groups `G_1` and `G_2` are cyclic groups of prime order `q = 21888242871839275222246405745257275088548364400416034343698204186575808495617`.
* `G_1` is defined on the curve `Y^2 = X^3 + 3` over the field `F_p` with `p = 21888242871839275222246405745257275088696311157297823662689037894645226208583` with generator `P1 = (1, 2)`.
* `G_2` is defined on the curve `Y^2 = X^3 + 3/(i+9)` over a different field `F_p^2 = F_p[i] / (i^2 + 1)` (`p` is the same as above) with generator
```js
P2 = (
  11559732032986387107991004021392285783925812861821192530917403151452391805634 * i +
  10857046999023057135944570762232829481370756359578518086990519993285655852781,
  4082367875863433681332203403145435568316851327593401208105741076214120093531 * i +
  8495653923123431417604973247489272438418190587263600148770280649306958101930
)
```
* `G_2` is the only group of order `q` of that elliptic curve over the field `F_p^2`
* Other generator of order `q` instead of `P2 would define the same `G_2`
* The concrete value of `P2` is useful for those who doubt the existence of a group of order `q`, just compare the concrete values of `q * P2` and `P2`.

## Encoding
* Elements of `F_p` are encoded as 32 byte big-endian numbers. An encoding value of `p` or larger is invalid.
* Elliptic curve points are encoded as a [Jacobian pair(may help)](https://arxiv.org/abs/math/0209159) `(X, Y)` where the point at infinity is encoded as `(0, 0)`.
* The number `k` is derived from the input length.
* Length of returned data is always exactly 32 bytes and encoded as a 32 byte big-endian number.

## Gas costs
* `80 000 * k + 100 000`, where `k` is the number of points or, equivalently, the length of the input divided by `192`.

## Rationale
(Same as EIP-196)
* Encoding of field elements in `F_p^2` was chosen in this order to be in line with the big endian encoding of the elements themselves.