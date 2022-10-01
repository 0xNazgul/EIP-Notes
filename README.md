# Ethereum Improvement Proposal Notes
I've put this off far too long and if your reading this you have too. I finally got sick of putting off sitting down and reading all proposals and taking notes on them (Review/Draft/Stagnant/Withdrawn coming soon). Some of it is word for word but I find typing them out myself helps it stick better. Use to just open them up to reference but no longer. I would start with [Meta](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#meta) category for they go through all hard forks, what EIPs were added and any other changes. Then go through EIPs not added via hard fork in [Core](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#core), [Networking](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#networking) and [Interface](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#interface). 

*Note*: I don't plan on doing notes on the [ERC section](https://eips.ethereum.org/erc). For most of them, no matter what when reviewing code I'll always have them up for referencing.

| Catagory | 
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Meta](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#meta) |
| [Core](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#core) |
| [Networking](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#networking) |
| [Interface](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#interface) |

# Meta
| Number | Title
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [EIP-606](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-606-Homestead.md) | Hardfork Meta: Homestead |
| [EIP-607](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-607-Spurious-Dragon.md) | Hardfork Meta: Spurious Dragon |
| [EIP-608](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-608-Tangerine-Whistle.md) | Hardfork Meta: Tangerine Whistle |
| [EIP-609](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-609-Byzantium.md) | Hardfork Meta: Byzantium |
| [EIP-779](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-779-DAO-Fork.md) | Hardfork Meta: DAO Fork |
| [EIP-1013](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-1013-Constantinople.md) | Hardfork Meta: Constantinople |
| [EIP-1679](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-1679-Istanbul.md) | Hardfork Meta: Istanbul |
| [EIP-1716](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-1716-Petersburg.md) | Hardfork Meta: Petersburg |
| [EIP-2387](https://github.com/0xNazgul/EIP-Notes/blob/main/Meta/EIP-2387-Muir-Glacier.md) | Hardfork Meta: Muir Glacier |


# Core
| Number | Title
|--------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| [EIP-2](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2-Homestead.md) | Homestead Hard-fork Changes |
| [EIP-5](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-5-RETURN-CALL.md) |	Gas Usage for `RETURN` and `CALL*` | 
| [EIP-7](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-7-DELEGATECALL.md)	| DELEGATECALL |
| [EIP-100](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-100-Change-difficulty-adjustment.md) |	Change difficulty adjustment to target mean block time including uncles |
| [EIP-140](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-140-REVERT-instruction.md) |	REVERT instruction |
| [EIP-141](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-141-invalid-EVM-instruction.md) |	Designated invalid EVM instruction |
| [EIP-145](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-145-Bitwise-shifting.md)	| Bitwise shifting instructions in EVM |
| [EIP-150](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-150-Gas-cost-changes.md) |	Gas cost changes for IO-heavy operations |
| [EIP-152](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-152-Add-BLAKE2.md) |	Add BLAKE2 compression function `F` precompile |
| [EIP-155](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-155-Simple-replay-attack-protection.md) |	Simple replay attack protection |
| [EIP-158](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-158-State-clearing.md) |	State clearing |
| [EIP-160](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-160-EXP-cost-increase.md) |	EXP cost increase |
| [EIP-161](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-161-State-trie-clearing.md) |	State trie clearing (invariant-preserving alternative) |
| [EIP-170](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-170-Contract-code-size-limit.md) |	Contract code size limit |
| [EIP-196](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-196-Precompiled-contracts-for-addition.md) |	Precompiled contracts for addition and scalar multiplication on the elliptic curve alt_bn128 |
| [EIP-197](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-197-Precompiled-contracts-for-optimal.md) |	Precompiled contracts for optimal ate pairing check on the elliptic curve alt_bn128 |
| [EIP-198](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-198-Big-integer-modular-exponentiation.md) |	Big integer modular exponentiation |
| [EIP-211](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-211-RETURNDATASIZE-RETURNDATACOPY.md) |	New opcodes: RETURNDATASIZE and RETURNDATACOPY |
| [EIP-214](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-214-STATICCALL.md) |	New opcode STATICCALL |
| [EIP-225](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-225-Clique-proof-of-authority.md) |	Clique proof-of-authority consensus protocol |
| [EIP-649](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-649-Metropolis-Difficulty-Bomb.md) |	Metropolis Difficulty Bomb Delay and Block Reward Reduction |
| [EIP-658](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-658-Embedding-transaction.md) |	Embedding transaction status code in receipts |
| [EIP-1014](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1014-Skinny-CREATE2.md) | Skinny CREATE2 |
| [EIP-1052](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1052-EXTCODEHASH.md) | EXTCODEHASH opcode |
| [EIP-1108](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1108-Reduce-alt_bn128.md) | Reduce alt_bn128 precompile gas costs |
| [EIP-1234](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1234-Constantinople-Difficulty-Bomb-Delay.md) | Constantinople Difficulty Bomb Delay and Block Reward Adjustment |
| [EIP-1283](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1283-Net-gas-metering.md) | Net gas metering for SSTORE without dirty maps |
| [EIP-1344](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1344-ChainID.md) | ChainID opcode |
| [EIP-1559](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1559-Fee-market-change.md) | Fee market change for ETH 1.0 chain |
| [EIP-1884](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-1884-Repricing-for-trie-size.md) | Repricing for trie-size-dependent opcodes |
| [EIP-2028](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2028-Transaction-data-gas.md) | Transaction data gas cost reduction |
| [EIP-2200](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2200-Structured-Definitions.md) | Structured Definitions for Net Gas Metering |
| [EIP-2384](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2384-Muir-Glacier-Difficulty.md) | Muir Glacier Difficulty Bomb Delay |
| [EIP-2565](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2565-ModExp-Gas-Cost.md) | ModExp Gas Cost |
| [EIP-2681](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2681-Limit-account-nonce.md) | Limit account nonce to 2^64-1 |
| [EIP-2718](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2718-Typed-Transaction-Envelope.md) | Typed Transaction Envelope |
| [EIP-2929](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2929-Gas-cost-increases.md) | Gas cost increases for state access opcodes |
| [EIP-2930](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-2930-Optional-access-lists.md) | Optional access lists |
| [EIP-3198](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-3198-BASEFEE-opcode.md) | BASEFEE opcode |
| [EIP-3529](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-3529-Reduction-in-refunds.md) | Reduction in refunds |
| [EIP-3541](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-3541-Reject-new-contract-code.md) | Reject new contract code starting with the 0xEF byte |
| [EIP-3554](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-3554-Difficulty-Bomb-Delay.md) | Difficulty Bomb Delay to December 2021 |
| [EIP-3607](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-3607-Reject-transactions.md) | Reject transactions from senders with deployed code |
| [EIP-3675](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-3675-Upgrade-consensus.md) | Upgrade consensus to Proof-of-Stake |
| [EIP-4345](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-4345-Difficulty-Bomb-Delay.md) | Difficulty Bomb Delay to June 2022 |
| [EIP-4399](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-4345-Difficulty-Bomb-Delay.md) | Supplant DIFFICULTY opcode with PREVRANDAO |
| [EIP-5133](https://github.com/0xNazgul/EIP-Notes/blob/main/Core/EIP-5133-Delaying-Difficulty-Bomb.md) | Delaying Difficulty Bomb to mid-September 2022 |

# Networking
| Number | Title
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [EIP-8](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-8-devp2p-Forward.md) | devp2p Forward Compatibility Requirements for Homestead |
| [EIP-627](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-627-Whisper-Specification.md) | Whisper Specification |
| [EIP-706](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-706-DEVp2p-snappy-compression.md) | DEVp2p snappy compression |
| [EIP-778](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-778-Ethereum-Node-Records.md) | Ethereum Node Records (ENR) |
| [EIP-868](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-868-Node-Discovery.md) | Node Discovery v4 ENR Extension |
| [EIP-2124](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-2124-Fork-identifier-for-chain.md) | Fork identifier for chain compatibility checks |
| [EIP-2364](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-2364-eth6-forkid-extended.md) | eth/64: forkid-extended protocol handshake |
| [EIP-2464](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-2464-eth65-transaction-announcements.md) | eth/65: transaction announcements and retrievals |
| [EIP-2976](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-2976-Typed-Transactions-over-Gossip.md) | Typed Transactions over Gossip |

# Interface
| Number | Title
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [EIP-6](https://github.com/0xNazgul/EIP-Notes/blob/main/Interface/EIP-6-Renaming-SUICIDE.md) | Renaming SUICIDE opcode |
| [EIP-234](https://github.com/0xNazgul/EIP-Notes/blob/main/Interface/EIP-234-Add-blockHash.md) | Add `blockHash` to JSON-RPC filter options |
| [EIP-695](https://github.com/0xNazgul/EIP-Notes/blob/main/Interface/EIP-695-Create-eth_chainId-method%20.md) | Create `eth_chainId` method for JSON-RPC |
| [EIP-712]() | Typed structured data hashing and signing |
| [EIP-1193]() | Ethereum Provider JavaScript API |
| [EIP-2159]() | Common Prometheus Metrics Names for Clients |
| [EIP-2696]() | JavaScript `request` method RPC transport |
| [EIP-2700]() | JavaScript Provider Event Emitter |
