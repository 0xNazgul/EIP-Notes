# EIP-649: Metropolis Difficulty Bomb Delay and Block Reward Reduction
Average block times are increasing from Ice Age and is slowly accelerating. This is meant to delay the bomb for a 1 1/2 years, reduce the block rewards (from Byzantium fork).
* The client will calculate the difficulty based on a fake block number suggesting the client that the difficulty bomb is adjusting around 3 million blocks later (previously Homestead fork). 
* Block rewards will be adjusted to a base of 3ETH with uncle and nephew rewards to be adjusted.

## Motivation
Casper development and switch to PoS are delayed, PoW should be able to hold the 15 second average. The idea is the block reward reduction will offset the ice age delay and leave the system in the same general state as before. 
* Will also decrease the likelihood of a miner driven chain split before PoS.

## Specification
**Relax Difficulty with Fake Block Number**

For the purposes of `calc_difficulty`, simply replace the use of `block.number`, as used in the exponential ice age component, with the formula:
`fake_block_number = max(0, block.number - 3_000_000) if block.number >= BYZANTIUM_FORK_BLKNUM else block.number`

**Adjust Block, Uncle, and Nephew rewards**

To ensure a constant Ether issuance, adjust the block reward to `new_block_reward` (3 ETH):
`new_block_reward = 3_000_000_000_000_000_000 if block.number >= BYZANTIUM_FORK_BLKNUM else block.reward`

If an uncle is included in a block for `block.number >= BYZANTIUM_FORK_BLKNUM` such that `block.number - uncle.number = k`, the uncle reward:
`new_uncle_reward = (8 - k) * new_block_reward / 8`

The nephew reward for `block.number >= BYZANTIUM_FORK_BLKNUM`:
`new_nephew_reward = new_block_reward / 32`

* Both are adjusted with the `new_block_reward`.
