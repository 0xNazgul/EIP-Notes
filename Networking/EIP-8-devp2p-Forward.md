# EIP-8: devp2p Forward Compatibility Requirements for Homestead
This will introduce forward-compatibility requirements for implementations of the `devp2p Wire Protocol`, the `RLPx Discovery Protocol` and the `RLPx TCP Transport Protocol`. Clients which implement this behave according to `Postel’s Law`:
`Be conservative in what you do, be liberal in what you accept from others.`

## Specification
1. Implementations of the `devp2p Wire Protocol` should ignore version number of hello packets. When sending hello packet, the version element should be set to the highest `devp2p` version supported. 
	* Should also ignore any additional list elements at the end of the hello packet.

2. Implementations of the `RLPx Discovery Protocol` should not validate the version number of the ping packet, ignore any additional list elements in any packet, and ignore any data after the first `RLP` value in any packet. 
	* Discovery packets with unknown packet type should be discarded silently. The maximum size of any discovery packet is still 1280 bytes.

3. Implementations of the `RLPx TCP Transport protocol` should accept a new encoding for the encrypted key establishment handshake packets. If an EIP-8 style `RLPx auth-packet` is received, the corresponding `ack-packet` should be sent using the rules below.
	* Decoding the `RLP` data in `auth-body and ack-body` should ignore mismatches of `auth-vsn and ack-vsn`, any additional list elements and any trailing data after the list. 
	* During the transitioning period, implementations should pad `auth-body` with at least 100 bytes of junk data. Adding a random amount in range [100, 300] is recommended to vary the size of the packet.
```
auth-vsn         = 4
auth-size        = size of enc-auth-body, encoded as a big-endian 16-bit integer
auth-body        = rlp.list(sig, initiator-pubk, initiator-nonce, auth-vsn)
enc-auth-body    = ecies.encrypt(recipient-pubk, auth-body, auth-size)
auth-packet      = auth-size || enc-auth-body

ack-vsn          = 4
ack-size         = size of enc-ack-body, encoded as a big-endian 16-bit integer
ack-body         = rlp.list(recipient-ephemeral-pubk, recipient-nonce, ack-vsn)
enc-ack-body     = ecies.encrypt(initiator-pubk, ack-body, ack-size)
ack-packet       = ack-size || enc-ack-body

where

X || Y
    denotes concatenation of X and Y.
X[:N]
    denotes an N-byte prefix of X.
rlp.list(X, Y, Z, ...)
    denotes recursive encoding of [X, Y, Z, ...] as an RLP list.
sha3(MESSAGE)
    is the Keccak256 hash function as used by Ethereum.
ecies.encrypt(PUBKEY, MESSAGE, AUTHDATA)
    is the asymmetric authenticated encryption function as used by RLPx.
    AUTHDATA is authenticated data which is not part of the resulting ciphertext,
    but written to HMAC-256 before generating the message tag.
```

