# EIP-2 Homestead Hard fork

## Specification

1. Creation of contracts (by TX) is now 53,000 instead of 21,000.
* Creation of contracts via `CREATE` is unaffected

2. TX signatures whose s-value > `secp256k1n/2` are now considered invalid. 
* ECDSA recover precompiled contract is unchanged (Useful for old Bitcoin signatures)

3. Contract creation now has an goes out-of-gas error when not funded enough instead of leaving an empty contract.

4. Difficulty adjustment algorithm goes from:
* `block_diff = parent_diff + parent_diff // 2048 * (1 if block_timestamp - parent_timestamp < 13 else -1) + int(2**((block.number // 100000) - 2))`
	* `int(2**((block.number // 100000) - 2))` is the difficulty adjustment 
* `block_diff = parent_diff + parent_diff // 2048 * max(1 - (block_timestamp - parent_timestamp) // 10, -99) + int(2**((block.number // 100000) - 2))`
	* `//` is the integer division

## Rationale

1. There was a way to create contracts via TX. The cost to do so was 21,000 where for contracts it is 32,000. On top of this on could use suicide refunds to make a simple ether value transfer only using 11,664 gas.

2. With s value being able to be `0 < s < secp256k1n` TX malleability was a concern by:
* Flipping the s value from s to `secp256k1n - s`
* Flipping the v `27 -> 28`, `28 -> 27` 

This would give a valid signature that creates Ui inconvenience. For an attacker can force the TX to get confirmed in a block with a different hash from the TX any user sends. Disrupting user interfaces that use TX hashes for tracking IDs. It's not that big of a security issue given:
* Ethereum uses addresses not TX hashes for input of ether value transfer

3. This allows for users to better handle out-of-gas contract creation errors and improves safer creation

4. Changes to the difficulty adjustment solve a problem with the excess number of miners mining blocks with a timestamp equal to `parent_timestamp + 1`. The issue this creates is:
* Skewed block time distribution so that the current block time algorithm (targets a 13 second **median**, (would still be untouched)) mean started to increase. 
* 51% miner attack doing this could have the mean increase infinitly.
