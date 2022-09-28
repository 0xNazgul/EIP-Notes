# EIP-778 Ethereum Node Records

## Specification
The components of a node record are:
* `signature`: cryptographic signature of record contents
* `seq`: The sequence number, a 64-bit unsigned integer. Nodes should increase the number whenever the record changes and republish the record.
* The remainder of the record consists of arbitrary key/value pairs

A record’s signature is made and validated according to an `identity scheme`. The `identity scheme` is also responsible for deriving a node’s address in the DHT.

The key/value pairs must be sorted by key and must be unique. The keys can technically be any byte sequence, but ASCII text is preferred. Key names in the table below have pre-defined meaning.
```
id: name of identity scheme, e.g. “v4”
secp256k1: compressed secp256k1 public key, 33 bytes
ip: IPv4 address, 4 bytes
tcp: TCP port, big endian integer
udp: UDP port, big endian integer
ip6: IPv6 address, 16 bytes
tcp6: IPv6-specific TCP port, big endian integer
udp6: IPv6-specific UDP port, big endian integer
```
All keys except `id` are optional, including IP addresses and ports. A record without endpoint information is still valid as long as its signature is valid. If no tcp6 / udp6 port is provided, the `tcp/udp` port applies to both IP addresses. Declaring the same port number in both `tcp, tcp6 or udp, udp6` should be avoided but doesn’t render the record invalid.

## RLP Encoding
The canonical encoding of a node record is an RLP list of `[signature, seq, k, v, ...]`. The maximum encoded size of a node record is 300 bytes and rejects records larger than this size.

Records are signed and encoded as follows:
```
content = [seq, k, v, ...]
signature = sign(content)
record = [signature, seq, k, v, ...]
```

## Text Encoding
The textual form of a node record is the base64 encoding of its RLP representation, prefixed by `enr:`. Implementations should use the [URL-safe base64 alphabet](https://www.rfc-editor.org/rfc/rfc4648#section-5) and omit padding characters.

## v4 Identity Scheme
This specification defines a single identity scheme to be used as the default until other schemes are defined by further EIPs.
* To sign record `content` with this scheme, apply the `keccak256` hash function (as used by the EVM) to `content`, then create a signature of the hash. The resulting 64-byte signature is encoded as the concatenation of the r and s signature values (the recovery ID v is omitted).
* To verify a record, check that the signature was made by the public key in the `secp256k1` key/value pair of the record.
* To derive a node address, take the `keccak256` hash of the uncompressed public key.