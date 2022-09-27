# EIP-4399: Supplant DIFFICULTY opcode with PREVRANDAO
Supplants the semantics of the return value of existing `DIFFICULTY 0x44` opcode and renames the opcode to `PREVRANDAO 0x44`. The return value of the `DIFFICULTY 0x44` instruction after this change is the output of the randomness beacon provided by the beacon chain.

## Motivation
App may benefit from using the randomness accumulated by the beacon chain. Thus, randomness outputs produced by the beacon chain should be accessible in the EVM. At the point of `TRANSITION_BLOCK` of the PoS upgrade, the `difficulty` block field must be `0` thereafter because there is no longer any PoW seal on the block. This means that the `DIFFICULTY 0x44` instruction no longer has it’s previous semantic meaning. Given prior analysis on the usage of `DIFFICULTY`, the value returned by the instruction mixed with other values is a common pattern used by smart contracts to obtain randomness. The instruction with the same number as the `DIFFICULTY` opcode returning outputs of the beacon chain `RANDAO` implementation makes the upgrade to PoS backwards compatible for existing smart contracts obtaining randomness from the `DIFFICULTY` instruction. Changes proposed allow for smart contracts to determine whether the upgrade to the PoS has already happened. This can be done by analyzing the return value of the `DIFFICULTY` instruction. A value greater than `2**64` indicates that the TX is being executed in the PoS block. Decompilers  and other similar tooling may also use this trick to discern the new semantics of the instruction if data of the block including the TX in question is available.

## Specification
`Block structure`: Beginning with `TRANSITION_BLOCK`, client software must set the value of the `mixHash`

`EVM`: Beginning with `TRANSITION_BLOCK`, the `DIFFICULTY 0x44` instruction must return the value of the `mixHash` field.
	*The gas cost of the `DIFFICULTY 0x44` opcode remains unchanged.

`Renaming`:
* The `mixHash` field should further be renamed to `prevRandao`.
* The `DIFFICULTY 0x44` opcode should further be renamed to `PREVRANDAO 0x44`.

## Security Considerations
### Biasability
The beacon chain `RANDAO` implementation gives every block proposer 1 bit of influence power per slot. Proposer may deliberately refuse to propose a block on the opportunity cost of proposer and TX fees to prevent beacon chain randomness from being updated in a particular slot.
* An effect of proposer’s influence power is limited in time and lasts until the first honest `RANDAO` reveal is made afterwards. This limitation does also exist in the case when proposers of `n` consecutive slots are colluding to get `n` bits of influence power. One honest block proposal is enough to unbiased the `RANDAO` even if it was biased during several slots in a row.
* The semantics of the `PREVRANDAO 0x44` instruction gives proposers another way to gain 1 bit of influence power on apps. Biased proposer may censor a rolling the dice TX to force it to be included into the next block, thus, force it to use a `RANDAO` mix that the proposer knows in advance. The opportunity cost in this case would be negligible.

### Predictability
Historical randomness provided by any decentralized oracle is 100% predictable. On the contrary, the randomness that is revealed in the future is predictable up to a limited extent.

A list of inputs influencing future randomness on the beacon chain consists of but is not limited to the following items:
* `Accumulated randomness` 
	* A `RANDAO` mix produced by the beacon chain in the last slot of epoch `N` is the main input to the function defining block proposers in each slot of epoch `N + MIN_SEED_LOOKAHEAD + 1`
* `Number of active validators`
	* A number of active validators throughout an epoch is another input to the block proposer function.
* `Effective balance` 
	* All else being equal, the lower the effective balance of a validator the lower the chance this validator has to be designated as a proposer in a slot.
* `Accidentally missed proposals`
	* Network conditions and other factors that are resulting in accidentally missed proposals is a source of highly qualitative entropy that impacts `RANDAO` mixes. Usual rate of missed proposals on the Mainnet is about `1%`.
	* These inputs may be predictable and malleable on a short range of slots but the longer the attempted lookahead the more entropy is accumulated by the beacon chain.

## Tips for app devs
These can reduce predictability and biasability of randomness outputs returned by `PREVRANDAO 0x44`:
1. Make your applications rely on the future randomness with a reasonably high lookahead. For example, an app stops accepting bids at the end of epoch `K` and uses a `RANDAO` mix produced in slot `K + N + E` to roll the dice, where `N` is a lookahead in epochs and `E` is a few slots into epoch `N + 1`.
2. At least four epochs of lookahead results in the following outcome:
	* A proposer set of epoch `N + 1` isn’t known at the end of epoch `K` breaking a direct link between bidders and dice rollers
	* A number of active validators is updated at the end of each epoch affecting a set of proposers of next epochs, thus, impacting a `RANDAO` mix used by the application to roll the dice
	* Due to Mainnet statistics, there is about a `100%` chance for the network to accidentally miss a proposal during this period of time which reduces predictability of a `RANDAO` mix used to roll the dice.
3. Setting `E` to a small number gives a third party a little time to gain influence power on the future randomness that is being used to roll the dice. This amount of time is defined by `MIN_SEED_LOOKAHEAD` parameter and is about 6 minutes on the Mainnet.
* A reasonably high distance between bidding and rolling the dice attempts to leave low chance for bidders controlling a subset of validators to directly exploit their influence power. Ultimately, this chance depends on the type of the game and on a number of controlled validators. For instance, a chance of a single validator to affect a one-time game is negligible, and becomes bigger for multiple validators in a repeated game scenario.