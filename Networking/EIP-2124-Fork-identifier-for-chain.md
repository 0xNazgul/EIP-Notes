# EIP-2124: Fork identifier for chain compatibility checks
Current nodes try to find each other by establishing random connection to remote machines looking like an Ethereum node, trying to find a useful peer. 
* Waste of time/resources
This focuses only on the definition of a summary, a generally useful fork identifier, and it's validation rules.

Currently the discovery protocol can't tell the difference between public and private networks. It checks by establishing a `TCP/IP` connection, wrap it with RLPx cryptography then executes an `eth` handshake. This is costly if it's on a different network. Also, even if the peer is on the same chain if there is a consensus upgrade and not everyone has upgraded it can not tell. This will help with a new identity scheme to summarize the chains current status solving:
* If two nodes are on different networks, they should never even consider connecting.
* If a hard fork passes, upgraded nodes should reject non-upgraded ones, but not before.
* If two chains share the same genesis, but not forks (ETH / ETC), they should reject each other.

## Specification
Each node maintains the following values:
* `FORK_HASH`: IEEE CRC32 checksum (`[4]byte`) of the genesis hash and fork blocks numbers that already passed.
	* The fork block numbers are fed into the CRC32 checksum in ascending order.
	* If multiple forks are applied at the same block, the block number is checksummed only once.
	* Block numbers are regarded as `uint64` integers, encoded in big endian format when checksumming.
	* If a chain is configured to start with a non-Frontier ruleset already in its genesis, that is not considered a fork.
* `FORK_NEXT`: Block number (`uint64`) of the next upcoming fork, or 0 if no next fork is known.

Validation rules:
1) If local and remote `FORK_HASH` matches, compare local head to `FORK_NEXT`.
	* The two nodes are in the same fork state currently. They might know of differing future forks, but that’s not relevant until the fork triggers (might be postponed, nodes might be updated to match).
		* 1a) A remotely announced but remotely not passed block is already passed locally, disconnect, since the chains are incompatible.
		* 1b) No remotely announced fork; or not yet passed locally, connect.
2) If the remote `FORK_HASH` is a subset of the local past forks and the remote `FORK_NEXT` matches with the locally following fork block number, connect.
	* Remote node is currently syncing. It might eventually diverge from us, but at this current point in time we don’t have enough information.
3) If the remote `FORK_HASH` is a superset of the local past forks and can be completed with locally known future forks, connect.
	* Local node is currently syncing. It might eventually diverge from the remote, but at this current point in time we don’t have enough information.
4) Reject in all other cases.
