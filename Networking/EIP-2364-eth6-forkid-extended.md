# EIP-2364: eth/64: forkid-extended protocol handshake
Specifies `forkid` as a new field in the Ethereum wire protocol (eth) handshake. This change is implemented as a new version of the wire protocol, `eth/64`.

## Specification
* Implement `forkid` generation and validation per EIP-2124.
* Advertise a new `eth` protocol capability (version) at `eth/64`
	* The old `eth/63` protocol should still be kept alive side-by-side, until `eth/64` is sufficiently adopted by implementors
* Redefine Status (0x00) for eth/64 to add a trailing `forkid` field:
	* `Old packet: [protocolVersion, networkId, td, bestHash, genesisHash]`
	* `New packet: [protocolVersion, networkId, td, bestHash, genesisHash, forkid]`, where `forkid` is `[forkHash: [4]byte, forkNext: uint64]`

Whenever two peers connect using the `eth/64` protocol, the updated `Status` message must be sent as the protocol handshake, and each peer must validate the remote `forkid`, disconnecting at a detected incompatibility.