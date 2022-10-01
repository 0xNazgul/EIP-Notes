# EIP-2159: Common Prometheus Metrics Names for Clients
Many clients expose a range of metrics in a format compatible with Prometheus to allow operators to monitor the client's behaviour and performance and raise alerts if the chain isn't progressing or there are other indications of errors. By standardizing the naming and format of these common metrics, operators are able to monitor the operation of multiple clients in a single dashboard or alerting configuration.

## Specification
The following expose metrics to Prometheus. Clients may expose additional metrics however these should not use the `ethereum_` prefix.
* `ethereum_blockchain_height`: The current height of the canonical chain with a JSON-RPC equivalent of `eth_blockNumber`
* `ethereum_best_known_block_number`: The estimated highest block available with a JSON-RPC equivalent of `highestBlock` of `eth_syncing or eth_blockNumber` if not syncing
* `ethereum_peer_count`: The current number of peers connected with a JSON-RPC equivalent of `net_peerCount`
* `ethereum_peer_limit`: The maximum number of peers this node allows to connect	