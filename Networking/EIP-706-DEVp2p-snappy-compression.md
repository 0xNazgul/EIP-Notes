# EIP-706 DEVp2p snappy compression 
The current DEVp2p used by Ethereum currently does not employ any form of compression. This results in a massive amount of bandwidth wasted in the entire network, making both initial sync as well as normal operation slower and more laggy.
* This proposes a tiny extension to the DEVp2p protocol to enable `Snappy compression` on all message payloads after the initial handshake. After extensive benchmarks, results show that data traffic is decreased by 60-80% for initial sync.

## Specification
Bump the advertised DEVp2p version number from 4 to 5. If during handshake, the remote side advertises support only for version 4, run the exact same protocol as until now.

If the remote side advertises a DEVp2p version >= 5, inject a Snappy compression step right before encrypting the DEVp2p message during sending:
* A message consists of {Code, Size, Payload}
* Compress the original payload with Snappy and store it in the same field.
* Update the message size to the length of the compressed payload.
* Encrypt and send the message as before, oblivious to compression.

Similarly to message sending, when receiving a DEVp2p v5 message from a remote node, insert a Snappy decompression step right after the decrypting the DEVp2p message:
* A message consists of {Code, Size, Payload}
* Decrypt the message payload as before, oblivious to compression.
* Decompress the payload with Snappy and store it in the same field.
* Update the message size to the length of the decompressed payload.

Important caveats:
* The handshake message is never compressed, since it is needed to negotiate the common version.
* Snappy framing is not used, since the DEVp2p protocol already message oriented.
	* Snappy supports uncompressed binary literals too, leaving room for fine-tuned future optimizations for already compressed or encrypted data that would have no gain of compression.

## Avoiding DOS attacks
* Currently a DEVp2p message length is limited to 24 bits, amounting to a maximum size of 16MB. 
* With the introduction of Snappy compression, care must be taken not to blindly decompress messages, since they may get significantly larger than 16MB.
	* However, Snappy is capable of calculating the decompressed size of an input message without inflating it in memory (the stream starts with the uncompressed length up to a maximum of `2^32 - 1` stored as a little-endian varint). This can be used to discard any messages which decompress above some threshold. The proposal is to use the same limit (16MB) as the threshold for decompressed messages. This retains the same guarantees that the current DEVp2p protocol does, so there won’t be surprises in application level protocols.