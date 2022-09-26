# EIP-3675: Upgrade consensus to Proof-of-Stake
Deprecates PoW and upgrades it with the new PoS consensus mechanism driven by the beacon chain. 

## Motivation
The beacon chain network has been running since December 2020. Neither safety nor liveness failures were detected during this period of time. This long period of running without failure demonstrates the sustainability of the beacon chain system and its readiness to become a security provider for the Ethereum Mainnet.

## Specification
* `PoW block`: Block that is built and verified by the existing PoW mechanism. 
* `PoS block`: Block that is built and verified by the new PoS mechanism.
* `Terminal PoW block`: A PoW block that satisfies the following conditions: `pow_block.total_difficulty >= TERMINAL_TOTAL_DIFFICULTY and pow_block.parent_block.total_difficulty < TERMINAL_TOTAL_DIFFICULTY`. 
	* There can be more than one terminal PoW block in the network
* `TERMINAL_TOTAL_DIFFICULTY`: The amount of total difficulty reached by the network that triggers the consensus upgrade. Ethereum Mainnet configuration `MUST` have this parameter set to the value `58750000000000000000000`.
* `TRANSITION_BLOCK`: The earliest PoS block of the canonical chain
* `POS_FORKCHOICE_UPDATED`: An event occurring when the state of the PoS fork choice is updated.
* `FORK_NEXT_VALUE`: A block number set to the `FORK_NEXT` parameter for the upcoming consensus upgrade.
* `TERMINAL_BLOCK_HASH`: Designates the hash of the terminal PoW block if set
* `TERMINAL_BLOCK_NUMBER`: Designates the number of the terminal PoW block if `TERMINAL_BLOCK_HASH` is set.

Events having the `POS_` prefix in the name (PoS events) are emitted by the new PoS consensus mechanism. They signify the corresponding assertion that has been made regarding a block specified by the event. The underlying logic of PoS events can be found in the beacon chain specification. On the occurrence of each PoS event the corresponding action that is specified by this EIP `MUST` be taken.

These must be taken into account when reading those parts of the specification that refer to the PoS events:
* Reference to a block that is contained by PoS events is provided in a form of a block hash unless another is explicitly specified.
* A `POS_FORKCHOICE_UPDATED` event contains references to the head of the canonical chain and to the most recent finalized block. Before the first finalized block occurs in the system the finalized block hash provided by this event is stubbed with `0x0000000000000000000000000000000000000000000000000000000000000000`.
* `FIRST_FINALIZED_BLOCK`: The first finalized block that is designated by `POS_FORKCHOICE_UPDATED` event and has the hash that differs from the stub.

## Client software configuration
The following are a part of client software configuration and `MUST` be included into its binary distribution:
* `TERMINAL_TOTAL_DIFFICULTY`
* `FORK_NEXT_VALUE`
* `TERMINAL_BLOCK_HASH`
* `TERMINAL_BLOCK_NUMBER`
	* If `TERMINAL_BLOCK_HASH` is stubbed with `0x0000000000000000000000000000000000000000000000000000000000000000` then `TERMINAL_BLOCK_HASH and TERMINAL_BLOCK_NUMBER` parameters `MUST NOT` take an effect.

## PoW block processing
PoW blocks that are descendants of any terminal PoW block `MUST NOT` be imported. This implies that a terminal PoW block will be the last PoW block in the canonical chain.
* `MAX_EXTRA_DATA_BYTES`: 32

## Block structure
Beginning with `TRANSITION_BLOCK`, a number of previously dynamic block fields are deprecated by enforcing these values to instead be constants. Each block field listed in the table below `MUST` be replaced with the corresponding constant value.

Field->Constant value->Comment
* `ommersHash`: `0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347`->`= Keccak256(RLP([]))`
* `difficulty`: `0`	 
* `mixHash`: `0x0000000000000000000000000000000000000000000000000000000000000000`
* `nonce`: `0x0000000000000000`
* `ommers`: `[]`->`RLP([]) = 0xc0`

Beginning with `TRANSITION_BLOCK`, the validation of the block’s `extraData` field changes: 
* The length of the block’s `extraData <= MAX_EXTRA_DATA_BYTES` bytes.
* Logic and validity conditions of block fields that are not specified here `MUST` remain unchanged. Additionally, the overall block format `MUST` remain unchanged.
* Subsequent EIPs may override the constant values specified above to provide additional functionality. 

## Block validity
Beginning with `TRANSITION_BLOCK`, the block validity conditions `MUST` be altered by the following:
* Remove verification of the block’s `difficulty` value with respect to the difficulty formula.
* Remove verification of the block’s `nonce and mixHash` values with respect to the Ethash function.
* Remove all validation rules that are evaluated over the list of ommers and each member of this list.
* Add verification of the fields noted in the block structure section.
	* If one of the new rules fails then the block `MUST` be invalidated.
	* Validity rules that are not specified in the list above `MUST` remain unchanged.

`Transition block validity`: `TRANSITION_BLOCK` must be a child of a terminal PoW block. That is, a parent of `TRANSITION_BLOCK` must satisfy terminal PoW block conditions.

## Block and ommer rewards
Beginning with `TRANSITION_BLOCK`, block and ommer rewards are deprecated. Following actions must be taken:
* Remove increasing the balance of the block’s `beneficiary` account by the block reward.
* Remove increasing the balance of the block’s `beneficiary` account by the ommer inclusion reward per each ommer.
* Remove increasing the balance of the ommer’s `beneficiary` account by the ommer block reward per each ommer.
	* TX fee mechanics affecting the block’s `beneficiary account must remain unchanged.

## Fork choice rule
If set, `TERMINAL_BLOCK_HASH` parameter affects the PoW heaviest chain rule in the following way:
* Canonical blockchain must contain a block with the hash defined by `TERMINAL_BLOCK_HASH` parameter at the height defined by `TERMINAL_BLOCK_NUMBER` parameter.
	* This rule is akin to block hash whitelisting functionality already present in client software implementations.

As of the first `POS_FORKCHOICE_UPDATED` event, the fork choice rule must be altered in the following way:
* Remove the existing PoW heaviest chain rule.
* Adhere to the new PoS LMD-GHOST rule.

The new PoS LMD-GHOST fork choice rule is specified as follows. On each occurrence of a P`OS_FORKCHOICE_UPDATED` event including the first one, the following actions must be taken:
* Consider the chain starting at genesis and ending with the head block nominated by the event as the canonical blockchain.
* Set the head of the canonical blockchain to the corresponding block nominated by the event.
* Beginning with the `FIRST_FINALIZED_BLOCK`, set the most recent finalized block to the corresponding block nominated by the event.

Changes to the block tree store that are related to the above actions must be applied atomically.
* This rule must be strictly enforced. Optimistic updates to the head `MUST NOT` be made. That is – if a new block is processed on top of the current head block, this new block becomes the new head if and only if an accompanying `POS_FORKCHOICE_UPDATED` event occurs.

## Network
`Fork identifier`: For the purposes of the EIP-2124 fork identifier, nodes implementing this EIP `MUST` set the `FORK_NEXT` parameter to the `FORK_NEXT_VALUE`.
 
`devp2p`: The networking stack `SHOULD NOT` send the following messages if they advertise the descendant of any terminal PoW block:
* `NewBlockHashes 0x01`
* `NewBlock 0x07`

Beginning with receiving the `FIRST_FINALIZED_BLOCK`, the networking stack must discard the following ingress messages:
* `NewBlockHashes 0x01`
* `NewBlock 0x07`

Beginning with receiving the finalized block next to the `FIRST_FINALIZED_BLOCK`, the networking stack must remove the handlers corresponding to the following messages:
* `NewBlockHashes 0x01`
* `NewBlock 0x07`

Peers that keep sending these messages after the handlers have been removed should be disconnected.
* The logic of message handlers that are not affected by this section must remain unchanged.