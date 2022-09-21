# EIP-234: Add `blockHash` to JSON-RPC filter options
Adding an option to `JSON-RPC` filter options (used by `eth_newFilter and eth_getLogs`) that allows specifying the block hash that should be included in the results.
* This allows clients to fetch logs for specific blocks. Resolving some issues that make it difficult to author robust clients due to chain reorgs, network connection and result set not containing enough details in the empty case.

## Specification
The filter options used by `eth_newFilter` would have an additional optional parameter named `blockHash` whose value is a single block hash. The node responding to the request would: 
1. Send back an error if the block hash was not found 
2. It would return the results matching the filter (normal) constrained to the block provided. 
* Similar to the `fromBlock and toBlock` filter options

## Rationale
A client that needs reliable notification of both log additions and log removals cannot achieve this while relying solely on subscription and filters.
* Solution is to provide a robust mechanism for internal block/log additional/removal so the client can maintain a blockchain internally and subscribe for new blocks.
* On a new block the client can reconcile their internal model with the new block to get in sync with the node.

With a reliable stream of block one can look at the bloom filter for the new block and if the block may have logs of interest. An issue arises when a re-org occurs between when the client receives the block and when the client fetches the logs for the block
* With the current set of filter options the client can only ask for logs by block number. So the logs they get back will be for a block that isn't the block they want the logs for but the block that was re-orged.
* By looking at the resulting logs themselves and identifying whether or not they are for the block hash requested would work.
	* BUT if the result set is an empty array then the client is in a situation where they don't know what block the results are for. So there is no decision the client can make that allows them a guarantee of recovery
	* This creates a problem if the block was only transiently re-orged out because this may come back before the next block poll. The client will not see the reorg, they can assume the empty logs were for the wrong block refetch them, but they may continue to get empty results putting them right back into the same situation.

