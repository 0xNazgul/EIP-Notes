# EIP-225: Clique proof-of-authority consensus protocol
Clique is a proof-of-authority consensus protocol. It shadows the design of Ethereum mainnet, so it can be added to any client with minimal effort.

## Motivation
The first official testnet was Morden (July 2015-> February 2017) due to accumulated junk. Ropsten was born with a clean slate, until February 2017, when malicious actors attacked the low PoW and gradually inflate theblock gas limits to 9 billion (from 4.7 million).
* These attacks happened for the network is only as secure as the computing capacity. Parity team decided to roll back a number of blocks and enact a soft-fork rule that disallows gas limits.

This works short term:
* Not elegant for Ethereum supposed to have dynamic block limits
* Not portable for other clients need to implement new fork logic themselves
* Not compatible with sync modes so fast and light clients are out of luck
* Just prolonging the attacks since junk can still be steadily pushed in ad infinitum

## Standardized proof-of-authority
At the time PoS was out of firm grasps so this solution may be outdated. PoA was simple to implement and embed into existing Ethereum clients and easy for sync tech to not have to add custom logic.

## Design constraints
There are two approaches to syncing a blockchain in general:
* Classical approach: take the genesis block and crunch through all the transactions one by one. This is tried and proven, but in Ethereum complexity networks quickly turns out to be very costly computationally.
* The other is to only download the chain of block headers and verify their validity, after which point an arbitrary recent state may be downloaded from the network and checked against recent headers.

The idea of the PoA scheme is that blocks may only be minted by trusted signers. Every block (or header) that a client sees can be matched against the list of trusted signers. The challenge here is how to maintain a trusted list that will change over time.
* Answer: store it in an contract! NO: fast, light and warp sync don't have access to the state during syncing.

* The protocol of maintaining the list of authorized signers must be fully contained in the block headers.
Answer: change the structure of the block headers so it drops the notions of PoW, and adds new fields to cater for voting! NO: changing core data structure in multiple implementations would be really bad for development, maintenance and security.

* The protocol of maintaining the list of authorized signers must fit fully into the current data models.
Best was to stick to currently available ones since the EVM can't be use for voting and header fields can't be changed.

## Repurposing header fields for signing and voting
The field of chose, is currently used solely as fun metadata and is the 32 byte `extra-data` section in block headers. This will be extended with 65 bytes with the purpose of `secp256k1` miner sig. It makes the `miner` section in block headers obsolete.
* Changing the length of a header field is smooth and clients wouldn't need custom logic.

This is enough to validate a chain but what of updating a dynamic list of signers. The `miner` field can be repurposed along with the `nonce` field to create a voting protocol:
1. During regular blocks, both of these fields would be set to zero.
2. If a signer wishes to enact a change to the list of authorized signers, it will:
* Set the `miner` to the signer it wishes to vote about
* Set the `nonce` to 0 or `0xff...f` to vote in favor of adding or kicking out

Clients syncing the chain can tally up the votes during block processing and maintain a dynamically changing list of signers.
* Every epoch transition flushes all pending votes. Furthermore, these epoch transitions can also act as stateless checkpoints containing the list of current authorized signers within the header extra-data.
* Clients only need to sync up based on a checkpoint hash without having to replay all the voting.

# Attack Vectors

## Attack vector: Malicious signer
A malicious user could get added to the list, or that a signer key/machine is compromised. Solution is that given a list of N signers any signer may only mint 1 block out of every K. Limiting damage and gives miners time to vote them out

## Attack vector: Censoring signer
A signer (or group) attempt to censor out blocks that vote on removing them from the list. Restriction of allowed minting frequency of signers to 1 out of N/2. Ensuring malicious signers need to control at least 51% signing accounts.

## Attack vector: Spamming signer
Malicious signer injecting new vote proposals inside every block they mint. Since nodes need to tally up all votes to create the actual list of signers, they need to track all votes through time. Without placing a limit on the vote window, which could grow slowly yet unbounded. Solution is to place a window of W blocks after which votes are considered stale (an epoch).

## Attack vector: Concurrent blocks


## Specification
Constants:
1. `EPOCH_LENGTH`: Number of blocks after which to checkpoint and reset the pending votes.
* Suggested `30000` for the testnet to remain analogous to the mainnet `ethash` epoch.
2. `BLOCK_PERIOD`: Minimum difference between two consecutive block’s timestamps.
* Suggested `15`s for the testnet to remain analogous to the mainnet `ethash` target.
3. `EXTRA_VANITY`: Fixed number of extra-data prefix bytes reserved for signer vanity.
* Suggested `32 bytes` to retain the current extra-data allowance and/or use.
4. `EXTRA_SEAL`: Fixed number of extra-data suffix bytes reserved for signer seal.
* `65 bytes` fixed as signatures are based on the standard `secp256k1` curve.
* Filled with zeros on genesis block.
5. `NONCE_AUTH`: Magic nonce number `0xffffffffffffffff` to vote on adding a new signer.
6. `NONCE_DROP`: Magic nonce number `0x0000000000000000` to vote on removing a signer.
7. `UNCLE_HASH`: Always `Keccak256(RLP([]))` as uncles are meaningless outside of PoW.
8. `DIFF_NOTURN`: Block score (difficulty) for blocks containing out-of-turn signatures.
* Suggested 1 since it just needs to be an arbitrary baseline constant.
9. `DIFF_INTURN`: Block score (difficulty) for blocks containing in-turn signatures.
* Suggested 2 to show a slight preference over out-of-turn signatures.

per-block constants:
1. `BLOCK_NUMBER`: Block height in the chain, where the height of the genesis is block 0.
2. `SIGNER_COUNT`: Number of authorized signers valid at a particular instance in the chain.
3. `SIGNER_INDEX`: Zero-based index of the block signer in the sorted list of current authorized signers.
4. `SIGNER_LIMIT`: Number of consecutive blocks out of which a signer may only sign one.
* Must be `floor(SIGNER_COUNT / 2) + 1` to enforce majority consensus on a chain.

Repurposed the `ethash` header fields:
1. `beneficiary/miner`: Address to propose modifying the list of authorized signers with.
* Should be filled with zeroes normally, modified only while voting.
* Arbitrary values are permitted nonetheless (even meaningless ones such as voting out non signers) to avoid extra complexity in implementations around voting mechanics.
* Must be filled with zeroes on checkpoint (i.e. epoch transition) blocks.
* Transaction execution must use the actual block signer (see `extraData`) for the `COINBASE` opcode and transaction fees must be attributed to the signer account.
2. `nonce`: Signer proposal regarding the account defined by the beneficiary field.
* Should be `NONCE_DROP` to propose deauthorizing beneficiary as an existing signer.
* Should be `NONCE_AUTH` to propose authorizing beneficiary as a new signer.
* Must be filled with zeroes on checkpoint (i.e. epoch transition) blocks.
* Must not take up any other value apart from the two above (for now).
3. `extraData`: Combined field for signer vanity, checkpointing and signer signatures.
First `EXTRA_VANITY` bytes (fixed) may contain arbitrary signer vanity data.
Last `EXTRA_SEAL` bytes (fixed) is the signer’s signature sealing the header.
Checkpoint blocks must contain a list of signers (`N*20 bytes`) in between, omitted otherwise.
The list of signers in checkpoint block extra-data sections must be sorted in ascending byte order.
4. `mixHash`: Reserved for fork protection logic, similar to the extra-data during the DAO.
Must be filled with zeroes during normal operation.
5. `ommersHash`: Must be `UNCLE_HASH` as uncles are meaningless outside of PoW.
6. `timestamp`: Must be at least the parent `timestamp + BLOCK_PERIOD`.
7. `difficulty`: Contains the standalone score of the block to derive the quality of a chain.
* Must be `DIFF_NOTURN` if `BLOCK_NUMBER % SIGNER_COUNT != SIGNER_INDEX`
* Must be `DIFF_INTURN` if `BLOCK_NUMBER % SIGNER_COUNT == SIGNER_INDEX`

## Authorizing a block
Authorizing a block for the network a signer needs to sign the blocks sighash containing everything except the sig itself. The sighash is signed using the standard `secp256k1` curve and the resulting 65 byte signature (`R, S, V, where V is 0 or 1`) is embedded into the `extraData` as the trailing 65 byte suffix.
* Each singer is allowed to sign maximum one out of `SIGNER_LIMIT` consecutive blocks.

## Authorization strategies
As long as signers conform to the specs, they can authorize and distribute blocks as seen fit. This strategy will reduce network traffic and small forks, so it’s a suggested feature:
* If a signer is allowed to sign a block
* Calculate the optimal signing time of the next block (`parent + BLOCK_PERIOD`).
* If the signer is in-turn, wait for the exact time to arrive, sign and broadcast immediately.
* If the signer is out-of-turn, delay signing by `rand(SIGNER_COUNT * 500ms)`.

## Voting on signers
Every epoch transition (genesis block included) acts as a stateless checkpoint, from which clients should be able to sync without requiring previous state. Meaning epoch headers must not contain votes, all non settled votes are discarded and tallying starts from scratch.

All non-epoch transition blocks:
* Signers may cast one vote per own block to propose a change to the authorization list.
* Only the latest proposal per target beneficiary is kept from a single signer.
* Votes are tallied live as the chain progresses (concurrent proposals allowed).
* Proposals reaching majority consensus `SIGNER_LIMIT` come into effect immediately.
* Invalid proposals are not to be penalized for client implementation simplicity.

A proposal coming into effect will discard all pending votes for that proposal and start with a clean slate.

## Cascading votes
A corner case can arise during signer deauthorization. When a previous signer is voted out, the number of signers required to approve a proposal may decrease by one. This might cause one or more pending proposals to reach majority consensus, the execution of which might cascade into new proposals passing.
* Handling this where the evaluation order might change the outcome of the final list. Since signers may invert their own votes in every block they mint, it’s not so obvious which proposal would be `first`.
* To avoid the pitfall from cascading executions would entail, the `Clique` proposal explicitly forbids cascading effects. If that causes other proposals to reach consensus, those will be executed when their respective beneficiaries are “touched” again.

## Voting strategies
* Since the blockchain can have small reorgs, a naive voting mechanism of “cast-and-forget” may not be optimal, since a block containing a singleton vote may not end up on the final chain.
* A working strategy is to allow users to configure “proposals” on the signers. The signing code can then pick a random proposal for every block it signs and inject it. This ensures that multiple concurrent proposals as well as reorgs get eventually noted on the chain.
* This list may be expired after a certain number of `blocks/epochs`, but it’s important to realize that “seeing” a proposal pass doesn’t mean it won’t get reorged, so it should not be immediately dropped when the proposal passes.

