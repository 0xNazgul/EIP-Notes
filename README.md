# Ethereum Improvement Proposal Notes
I've put this off far too long and if your reading this you have too. I finally got sick of putting off sitting down and reading all proposals and taking notes on them (Review/Draft/Stagnant/Withdrawn coming soon). Some of it is word for word but I find typing them out myself helps it stick better. Use to just open them up to reference but no longer. I would start with [Meta](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#meta) category for they go through all hard forks, what EIPs were added and any other changes. Then go through EIPs not added via hard fork in [Core](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#core), [Networking](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#networking) and [Interface](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#interface). Lastly, [ERC](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#erc) or if you don't care about the fun facts and how Ethereum became what it is today just read these. 

| Catagory | 
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Core](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#core) |
| [Networking](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#networking) |
| [Interface](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#interface) |
| [ERC](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#erc) |
| [Meta](https://github.com/0xNazgul/EIP-Notes/edit/main/README.md#meta) |


# Core
| Number | Title
|--------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| [EIP-2]() | Homestead Hard-fork Changes |
| [EIP-5]() |	Gas Usage for `RETURN` and `CALL*` | 
| [EIP-7]()	| DELEGATECALL |
| [EIP-100]() |	Change difficulty adjustment to target mean block time including uncles |
| [EIP-140]() |	REVERT instruction |
| [EIP-141]() |	Designated invalid EVM instruction |
| [EIP-145]()	| Bitwise shifting instructions in EVM |
| [EIP-150]() |	Gas cost changes for IO-heavy operations |
| [EIP-152]() |	Add BLAKE2 compression function `F` precompile |
| [EIP-155]() |	Simple replay attack protection |
| [EIP-158]() |	State clearing |
| [EIP-160]() |	EXP cost increase |
| [EIP-161]() |	State trie clearing (invariant-preserving alternative) |
| [EIP-170]() |	Contract code size limit |
| [EIP-196]() |	Precompiled contracts for addition and scalar multiplication on the elliptic curve alt_bn128 |
| [EIP-197]() |	Precompiled contracts for optimal ate pairing check on the elliptic curve alt_bn128 |
| [EIP-198]() |	Big integer modular exponentiation |
| [EIP-211]() |	New opcodes: RETURNDATASIZE and RETURNDATACOPY |
| [EIP-214]() |	New opcode STATICCALL |
| [EIP-225]() |	Clique proof-of-authority consensus protocol |
| [EIP-649]() |	Metropolis Difficulty Bomb Delay and Block Reward Reduction |
| [EIP-658]() |	Embedding transaction status code in receipts |
| [EIP-1014]() | Skinny CREATE2 |
| [EIP-1052]() | EXTCODEHASH opcode |
| [EIP-1108]() | Reduce alt_bn128 precompile gas costs |
| [EIP-1234]() | Constantinople Difficulty Bomb Delay and Block Reward Adjustment |
| [EIP-1283]() | Net gas metering for SSTORE without dirty maps |
| [EIP-1344]() | ChainID opcode |
| [EIP-1559]() | Fee market change for ETH 1.0 chain |
| [EIP-1884]() | Repricing for trie-size-dependent opcodes |
| [EIP-2028]() | Transaction data gas cost reduction |
| [EIP-2200]() | Structured Definitions for Net Gas Metering |
| [EIP-2384]() | Muir Glacier Difficulty Bomb Delay |
| [EIP-2565]() | ModExp Gas Cost |
| [EIP-2681]() | Limit account nonce to 2^64-1 |
| [EIP-2718]() | Typed Transaction Envelope |
| [EIP-2929]() | Gas cost increases for state access opcodes |
| [EIP-2930]() | Optional access lists |
| [EIP-3198]() | BASEFEE opcode |
| [EIP-3529]() | Reduction in refunds |
| [EIP-3541]() | Reject new contract code starting with the 0xEF byte |
| [EIP-3554]() | Difficulty Bomb Delay to December 2021 |
| [EIP-3607]() | Reject transactions from senders with deployed code |
| [EIP-4345]() | Difficulty Bomb Delay to June 2022 |
| [EIP-5133]() | Delaying Difficulty Bomb to mid-September 2022 |

# Networking
| Number | Title
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [EIP-8]() | devp2p Forward Compatibility Requirements for Homestead |
| [EIP-627]() | Whisper Specification |
| [EIP-706]() | DEVp2p snappy compression |
| [EIP-778]() | Ethereum Node Records (ENR) |
| [EIP-868]() | Node Discovery v4 ENR Extension |
| [EIP-2124]() | Fork identifier for chain compatibility checks |
| [EIP-2364]() | eth/64: forkid-extended protocol handshake |
| [EIP-2464]() | eth/65: transaction announcements and retrievals |
| [EIP-2976]() | Typed Transactions over Gossip |

# Interface
| Number | Title
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [EIP-6]() | Renaming SUICIDE opcode |
| [EIP-234]() | Add `blockHash` to JSON-RPC filter options |
| [EIP-695]() | Create `eth_chainId` method for JSON-RPC |
| [EIP-712]() | Typed structured data hashing and signing |
| [EIP-1193]() | Ethereum Provider JavaScript API |
| [EIP-2159]() | Common Prometheus Metrics Names for Clients |
| [EIP-2696]() | JavaScript `request` method RPC transport |
| [EIP-2700]() | JavaScript Provider Event Emitter |

# ERC
| Number | Title
|--------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| [EIP-20]() | Token Standard |
| [EIP-55]() | Mixed-case checksum address encoding |
| [EIP-137]() | Ethereum Domain Name Service - Specification |
| [EIP-162]() | Initial ENS Hash Registrar |
| [EIP-165]()	| Standard Interface Detection |
| [EIP-173]() | Contract Ownership Standard |
| [EIP-181]() | ENS support for reverse resolution of Ethereum addresses |
| [EIP-190]() | Ethereum Smart Contract Packaging Standard |
| [EIP-191]() | Signed Data Standard |
| [EIP-600]() | Ethereum purpose allocation for Deterministic Wallets |
| [EIP-601]() | Ethereum hierarchy for deterministic wallets |
| [EIP-681]() | URL Format for Transaction Requests |
| [EIP-721]() | Non-Fungible Token Standard |
| [EIP-777]() | Token Standard |
| [EIP-820]() | Pseudo-introspection Registry Contract |
| [EIP-1155]() | Multi Token Standard |
| [EIP-1167]() | Minimal Proxy Contract |
| [EIP-1271]() | Standard Signature Validation Method for Contracts |
| [EIP-1363]() | Payable Token |
| [EIP-1820]() | Pseudo-introspection Registry Contract |
| [EIP-1967]() | Proxy Storage Slots |
| [EIP-2098]() | Compact Signature Representation |
| [EIP-2309]() | ERC-721 Consecutive Transfer Extension |
| [EIP-2678]() | Revised Ethereum Smart Contract Packaging Standard (EthPM v3) |
| [EIP-2981]() | NFT Royalty Standard |
| [EIP-3156]() | Flash Loans |
| [EIP-3448]() | MetaProxy Standard |
| [EIP-3475]() | Abstract Storage Bonds |
| [EIP-3525]() | Semi-Fungible Token |
| [EIP-3668]() | CCIP Read: Secure offchain data retrieval |
| [EIP-4400]() | EIP-721 Consumable Extension |
| [EIP-4626]() | Tokenized Vaults |
| [EIP-4906]() | EIP-721 Metadata Update Extension |
| [EIP-4907]() | Rental NFT, an Extension of EIP-721 |
| [EIP-5192]() | Minimal Soulbound NFTs |
| [EIP-5313]() | Light Contract Ownership |

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
