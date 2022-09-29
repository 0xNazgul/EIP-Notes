# EIP-2464: eth/65: transaction announcements and retrievals
Introduces 3 new message types to announce a set of TXs without their content. 
* This permits reducing the bandwidth used for TX propagation from linear complexity in the number of peers to square root
* Reduces the initial TX exchange from 10s-100s MB to `len(pool) * 32B ~= 128KB`

##  Specification
3 new message types:
1. `NewPooledTransactionHashes 0x08: [hash_0: B_32, hash_1: B_32, ...]`
	* Specify one or more TXs that have appeared in the network and which have not yet been included in a block. To be maximally helpful, nodes should inform peers of all TXs that they may not be aware of.
	* There is no protocol violating hard cap on the number of hashes a node may announce to a remote peer, but 4_096 seems a sane chunk (128KB) to avoid a single packet hogging a network connection.
	* Nodes should only announce hashes of TXs that the remote peer could reasonably be considered not to know, but it is better to be over zealous than to have a nonce gap in the pool.
2. `GetPooledTransactions 0x09: [hash_0: B_32, hash_1: B_32, ...]`
	* Specify one or more TXs to retrieve from a remote peer’s TX pool.
	* There is no protocol violating hard cap on the number of TXs a node may request from a remote peer, but the recipient may enforce an arbitrary cap on the reply, which must not be considered a protocol violation. To keep wasted bandwidth down (unanswered hashes), 256 seems like a sane upper limit.
3. `PooledTransactions 0x0a: [[nonce: P, receivingAddress: B_20, value: P, ...], ...]`
	* Specify TXs from the local TX pool that the remote node requested via a `GetPooledTransactions 0x09` message. The items in the list are TXs in the format described in the main Ethereum specification.
	* The TXs must be in same order as in the request, but it is ok to skip TXs that are not available. This way if the response size limit is reached, requesters will know which hashes to request again (everything from the last returned TX) and which to assume unavailable (all gaps before the last returned TX).
	* A peer may respond with an empty reply iff none of the hashes match TXs in its pool. It is allowed to announce a TX that will not be served later if it gets included in a block in between.