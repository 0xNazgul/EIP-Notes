# EIP-2696: JavaScript `request` method RPC transport
Provides the description of an object that is made available to JavaScript applications which they can use to communicate with the Ethereum blockchain through (only describes the transport mechanism).

## Interface
```
interface RequestArguments {
	readonly method: string;
	readonly params?: readonly unknown[] | object;
}

interface EthereumProvider {
	request(args: RequestArguments): Promise<unknown>
}
```
The Provider must implement a `request` method on the exposed `EthereumProvider` object. The `request` method must be callable with a single parameter which contains the arguments for the request as defined in the interface above.

If the Provider supports a [JSON-RPC](https://www.jsonrpc.org/specification) then it must accept a `request` call for that JSON-RPC method with the `RequestArguments.method` argument matching the JSON-RPC `method` string for the RPC call and the `RequestArguments.params` matching the `params` object of the RPC call. 
* The `RequestArguments.params` should be encoded as a JavaScript object matching the specified JSON-RPC type, not encoded as a JSON string as would normally be the case when transporting JSON-RPC.

### Results
If the Provider supports a JSON-RPC request as specified elsewhere, then it must return an object that matches the expected `result` definition for the associated JSON-RPC request.

### Errors
```
interface ProviderRpcError extends Error {
	message: string;
	code: number;
	data?: unknown;
}
```
* `4001 User Rejected Request`: The user rejected the request.
* `4100 Unauthorized`: The requested method and/or account has not been authorized by the user.
* `4200	Unsupported Method`: The Provider does not support the requested method.
* `4900	Disconnected`: The Provider is disconnected from all chains.
*` 4901 Chain Disconnected`: The Provider is not connected to the requested chain.
If the Provider is unable to fulfill a request for any reason, it must resolve the promise as an error. The resolved error must be shaped as a `ProviderRpcError` defined above whenever possible. 
* If a `code` is provided that is listed in the list above, or in the [JSON-RPC](https://www.jsonrpc.org/specification#error_object), or in the associated JSON-RPC request standard being followed, then the error reason must align with the established meaning of that code and the `message` must match the provided `message`
* The `data` field may contain any data that is relevant to the error or would help the user understand or troubleshoot the error.
