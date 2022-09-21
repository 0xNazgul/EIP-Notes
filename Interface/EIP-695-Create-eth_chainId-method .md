# EIP-695 Create `eth_chainId` method for JSON-RPC
Include `eth_chainId` method in `eth_`-namespaced JSON-RPC methods. This method should return a single `STRING` result for an integer value in hexadecimal format, describing the currently configured `CHAIN_ID` value.

## Specification
`eth_chainId` returns the currently configured `CHAIN_ID` and should always correspond to the info in the current know head block.
* Ensures that caller of this RPC method can always use the retrieved info to sign TX built on top of the head.
