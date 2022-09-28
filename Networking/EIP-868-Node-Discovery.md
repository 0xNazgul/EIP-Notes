# EIP-868: Node Discovery v4 ENR Extension
Extension to Node Discovery Protocol v4 to enable authoritative resolution of [Ethereum Node Records (ENR)](https://github.com/0xNazgul/EIP-Notes/blob/main/Networking/EIP-778-Ethereum-Node-Records.md).

## Specification
### Ping Packet 0x01
`packet-data = [version, from, to, expiration, enr-seq]`
* `enr-seq`: current sequence number of the sending noede's record

### Pong Packet 0x02
`packet-data = [to, ping-hash, expiration, enr-seq]`

### ENRRequest Packet 0x05
`packet-data = [ expiration ]`
* When received the node should reply with an ENRResponse packet with the current version of its record
* Simplification  attacks guard: the sender of ENRResquest should have replied to a ping packet recently. The `expiration` field (UNIX timestamp) should be handled.

### ENRResponse Packet 0x06
`packet-data = [ request-hash, ENR ]`
* `request-hash`: The hash of the entire ENRRequest packet being replied to
* `ENR`: The node record
recipient should verify that the node record is signed by node who sent ENRResponse