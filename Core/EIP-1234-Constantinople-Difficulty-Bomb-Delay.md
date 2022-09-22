# EIP-1234: Constantinople Difficulty Bomb Delay and Block Reward Adjustment
Same reasoning as the last Ice Age delays this time should push it back 12 months and reduce the block rewards again. 
* Same Abstract BUT the client that the difficulty bomb is adjusting around 5 million blocks later and block rewards will be adjusted to a base of 2 ETH.

## Motivation
Same as last Ice Age delay

## Specification
All calculations are the same as last Ice Age delay. BUT for `new_block_reward`:
`new_block_reward = 2_000_000_000_000_000_000 if block.number >= CNSTNTNPL_FORK_BLKNUM else block.reward`