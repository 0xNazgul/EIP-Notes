# EIP-1108: Reduce alt_bn128 precompile gas costs
The elliptic curve arithmetic are currently expensive. lowering them would greatly help a lot of privacy solutions and scaling solutions.
* Changes to the underlying library by Go reference led to significant performance gains for `ECADD, ECMUL` and pairing check precompiled contracts on `alt_bn128` elliptic curve.
* Parity client field operations used by the precompile algorithms were optimized and change to pairing algo used by the `bn` crate brought speedups.
* Faster operations on Ethereum clients should see reduced gas costs.

## Specification
Gas changes:
* `ECADD 0x06`: 500->150
* `ECMUL 0x07`:	40_000->6_000
* Pairing check	`0x08`:	(80_000 * k + 100_000)->(34_000 * k + 45_000) (NOTE: k is the number of pairings)

`zk-SNARK` based protocols will not only reduce the gas cost to verify them but can also aid in [batching multiple proofs](https://github.com/matter-labs/Groth16BatchVerifier). Can also do another technique that can be used to split up `monolithic zk-SNARK circuits` into a batch of `zk-SNARK`s with smaller circuit sizes.
