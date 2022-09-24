# EIP-2028: Transaction data gas cost reduction
Reduce the gas cost of Calldata (`GTXDATANONZERO`) from its current value of 68 gas per byte to 16 gas per byte. 

## Motivation
Will allow for higher bandwidth of Calldata improves scalability as more data can fit within a single block.
* Layer two scalability: Layer two scaling solution can improve scalability by moving storage and computation off-chain.
	* Proof systems like STARKs use a single proof that attests to the computational integrity of a large computation.
	* Some solutions use fraud proofs which requires a transmission of merkle proofs
	* One optional data availability solution to layer two is to place data on the main chain via calldata
* Stateless clients: will be used to determine the price of the state access for the stateless client regime, which will be proposed in the State Rent. It is expected that the gas cost of state accessing operation will increase roughly proportional to the extra bandwidth required to transmit the block proofs as well as extra processing required to verify those block proofs.

## Rationale
This introduces a model with natural parameters:
* `lambda` denotes the block creation rate [1/s]: We treat the process of finding a PoW solution as a poisson process with rate lambda.
* `beta` - chain growth rate [1/s]: the rate at which new blocks are added to the heaviest chain.
* `D` - block delay [s]: The time that elapses between the mining of a new block and its acceptance by all the miners (all miners switched to mining on top of that block).

## Beta Lower Bound
`lambda => beta` because not all blocks that are found will enter the main chain (uncles). It was shown that for a blockchain using the longest chain rule, one may bound `beta` from below by `lambda/ (1+ D * lambda)`. This lower bound holds in extreme cases where the topology of the network is a clique in which the delay between each pair of nodes is `D`, the maximal possible delay. 

Recording both bounds on `beta`:
`_lambda_ >= _beta_ >= _lambda_ / (1 + D * _lambda_)               (*)`

## Security of the network
An attacker attempting to reorganize the main chain needs to generate blocks at a rate that is greater than `beta`. Fixing the difficulty level of the PoW puzzle, the total hash rate in the system is correlated to `lambda`. Thus, `beta / lambda` is defined as the efficiency of the system, as it measures the fraction of total hash power that is used to generate the main chain of the network.
Rearranging `*` gives this lower bound on efficiency in terms of delay:
`_beta_ / _lambda_ >= 1 / (1 + D * _lambda_)                 (**)`

## The delay parameter D
The network delay depends on the location of the mining node within the network and on the current network topology, and consequently is somewhat difficult to measure directly. Previously shown, propagation time scales with blocksize, and Vitalik Buterin showed that uncle rate, which is tightly related to efficiency `(**)` measure, also scales with block size.

The delay function can be decomposed into two parts `D = D_t + D_p`, where `D_t` is the delay caused by the transmission of the block and `D_p` is the delay caused by the processing of the block by the node. This model and tests will examine the effect of Calldata on each of `D_t` and `D_p`, postulating that their effect is different. This may be particularly relevant for Layer 2 Scalability and for Stateless Clients because most of the Calldata associated with these goals are Merkle authentication paths that have a large `D_t` component but relatively small `D_p` values.