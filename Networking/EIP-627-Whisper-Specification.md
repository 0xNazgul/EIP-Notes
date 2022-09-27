# EIP-627 Whisper Specification
Describes the format of Whisper messages within the `DEVp2p Wire Protocol`. This should substitute the existing [specification](https://github.com/ethereum/wiki/wiki/Whisper-Wire-Protocol) and here is more detailed [documentation](https://github.com/ethereum/go-ethereum/wiki/Whisper).

## Specification
All Whisper messages sent as `DEVp2p Wire Protocol` packets should be RLP-encoded arrays of data containing two objects, integer packet code followed by another object (whose type depends on the packet code).
* If Whisper node does not support a particular packet code, it should just ignore the packet without generating any error.

## Packet Codes
The message codes reserved for Whisper protocol are `0-127`. Messages with unknown codes must be ignored, for forward compatibility of future versions.

The Whisper sub-protocol should support the following packet codes:
```
Status: 0
Messages: 1
PoW Requirement: 2
Bloom Filter: 3
P2P Request: 126
P2P Message: 127
```
* P2P Request & P2P Message are optional but are reserved

## Packet Format and Usage
`Status [0]`
This packet contains two objects: 
1. Integer message code (0x00) 
2. Followed by a list of values: integer version, float PoW requirement, and bloom filter, in this order. 

* The bloom filter parameter is optional; if it is missing or nil, the node is considered to be full node.
* Status message should be sent after the initial handshake and prior to any other messages.

`Messages [1, whisper_envelopes]`
This packet contains two objects: 
1. Integer message code (0x01) 
2. Followed by a list (possibly empty) of Whisper Envelopes.

* This packet is used for sending the standard Whisper envelopes.

`PoW Requirement [2, PoW]`
This packet contains two objects:
1. Integer message code (0x02) 
2. Followed by a single floating point value of PoW. 

* This value is the `IEEE 754` binary representation of 64-bit floating point number. Values of `qNAN, sNAN, INF and -INF` and negative values  are not allowed.
* This packet is used by Whisper nodes for dynamic adjustment of their individual PoW requirements. Recipient of this message should no longer deliver the sender messages with PoW lower than specified in this message.
* PoW is defined as average number of iterations, required to find the current BestBit (the number of leading zero bits in the hash), divided by message size and TTL:
`PoW = (2**BestBit) / (size * TTL)`

PoW calculation:
```
fn short_rlp(envelope) = rlp of envelope, excluding env_nonce field.
fn pow_hash(envelope, env_nonce) = sha3(short_rlp(envelope) ++ env_nonce)
fn pow(pow_hash, size, ttl) = 2**leading_zeros(pow_hash) / (size * ttl)
where size is the size of the full RLP-encoded envelope.
```

`Bloom Filter [3, bytes]`
This packet contains two objects: 
1. Integer message code (0x03) 
2. Followed by a byte array of arbitrary size.
* This packet is used by Whisper nodes for sharing their interest in messages with specific topics.

The Bloom filter is used to identify a number of topics to a peer without compromising too much privacy over precisely what topics are of interest. Precise control over the information content may be maintained through the addition of bits. Blooms are formed by the bitwise `OR` operation on a number of bloomed topics. The bloom function takes the topic and projects them onto a 512-bit slice. At most, three bits are marked for each bloomed topic. The projection function is defined as a mapping from a 4-byte slice S to a 512-bit slice D; for ease of explanation, S will dereference to bytes, whereas D will dereference to bits.
```
LET D[*] = 0
FOREACH i IN { 0, 1, 2 } DO
LET n = S[i]
IF S[3] & (2 ** i) THEN n += 256
D[n] = 1
END FOR
```

`P2P Request [126, whisper_envelope]`
This packet contains two objects: 
1. Integer message code (0x7E) 
2. Followed by a single Whisper Envelope.

* This packet is used for sending Dapp-level peer-to-peer requests

`P2P Message [127, whisper_envelope]`
This packet contains two objects: 
1. Integer message code (0x7F) 
2. Followed by a single Whisper Envelope.

* This packet is used for sending the peer-to-peer messages, which are not supposed to be forwarded any further. 

## Whisper Envelope
Envelopes are RLP-encoded structures of the following format:
`[ Expiry, TTL, Topic, Data, Nonce ]`

* `Expiry`: 4 bytes (UNIX time in seconds).
* `TTL`: 4 bytes (time-to-live in seconds).
* `Topic`: 4 bytes of arbitrary data.
* `Data`: byte array of arbitrary size (contains encrypted message).
* `Nonce`: 8 bytes of arbitrary data (used for PoW calculation).

## Contents of Data Field of the Message (Optional)
This section outlines the optional description of Data Field to set up an example. 
* It is only relevant if you want to decrypt the incoming message, but if you only want to send a message, any other format would be perfectly valid and must be forwarded to the peers.
* Data field contains encrypted message of the Envelope. 
	* In case of symmetric encryption, it also contains appended Salt (AES Nonce, 12 bytes). Plaintext (unencrypted) payload consists of the following concatenated fields (in this sequence):
```
flags: 1 byte; first two bits contain the size of auxiliary field, third bit indicates whether the signature is present.
auxiliary field: up to 4 bytes; contains the size of payload.
payload: byte array of arbitrary size (may be zero).
padding: byte array of arbitrary size (may be zero).
signature: 65 bytes, if present.
salt: 12 bytes, if present (in case of symmetric encryption).
```
* Those unable to decrypt the message data are also unable to access the signature. The signature, if provided, is the ECDSA signature of the Keccak-256 hash of the unencrypted data using the secret key of the originator identity. The signature is serialised as the concatenation of the R, S and V parameters of the SECP-256k1 ECDSA signature, in that order. R and S are both big-endian encoded, fixed-width 256-bit unsigned. V is an 8-bit big-endian encoded, non-normalised and should be either 27 or 28.
* The padding field was introduced in order to align the message size, since message size alone might reveal important metainformation. Padding can be arbitrary size. 
	* However, it is recommended that the size of Data Field (excluding the Salt) before encryption should be factor of 256 bytes.

## Payload Encryption
* Asymmetric encryption uses the standard Elliptic Curve Integrated Encryption Scheme with SECP-256k1 public key.
* Symmetric encryption uses AES GCM algorithm with random 96-bit nonce.