# EIP-1193 Ethereum Provider JavaScript API 

## Specification
The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [RFC-2119](https://www.ietf.org/rfc/rfc2119.txt).

### Definitions
* Provider
	* A JavaScript object made available to a consumer, that provides access to Ethereum by means of a Client.
* Client
	* An endpoint that receives Remote Procedure Call (RPC) requests from the Provider, and returns their results.
* Wallet
	* An end-user application that manages private keys, performs signing operations, and acts as a middleware between the Provider and the Client.
* Remote Procedure Call (RPC)
	* A Remote Procedure Call (RPC), is any request submitted to a Provider for some procedure that is to be processed by a Provider, its Wallet, or its Client.

### Connectivity
The Provider is said to be “connected” when it can service RPC requests to at least one chain and “disconnected” when it cannot service RPC requests to any chain at all.

### API
The Provider must implement and expose the API defined in this section. All API entities must adhere to the types and interfaces defined in this section.

### Request
```
interface RequestArguments {
  readonly method: string;
  readonly params?: readonly unknown[] | object;
}

Provider.request(args: RequestArguments): Promise<unknown>;
```
The Provider must identify the requested RPC method by the value of `RequestArguments.method`. 
* If the requested RPC method takes any parameters, the Provider must accept them as the value of `RequestArguments.params`.

RPC requests must be handled such that the returned Promise either resolves with a value per the requested RPC method’s specification, or rejects with an error.
* If resolved, the Promise must resolve with a result per the RPC method’s specification. The Promise must not resolve with any RPC protocol-specific response objects, unless the RPC method’s return type is so defined.
* If the returned Promise rejects, it must reject with a `ProviderRpcError` as specified in the RPC Errors section below.

The returned Promise must reject if any of the following conditions are met:
* An error is returned for the RPC request.
	* If the returned error is compatible with the `ProviderRpcError` interface, the Promise may reject with that error directly.
* The Provider encounters an error or fails to process the request for any reason.
The returned Promise should reject if any of the following conditions are met:
* The Provider is disconnected.
	* If rejecting for this reason, the Promise rejection error code must be 4900.
* The RPC request is directed at a specific chain, and the Provider is not connected to that chain, but is connected to at least one other chain.
	* If rejecting for this reason, the Promise rejection error code must be 4901.

### Supported RPC Methods
Is any RPC method that may be called via the Provider.
* All supported RPC methods must be identified by unique strings.
* Providers may support whatever RPC methods required to fulfill their purpose, standardized or otherwise.

If an RPC method defined in a finalized EIP is not supported, it should be rejected with a `4200` error or an appropriate error per the RPC method’s specification.

### RPC Errors
```
interface ProviderRpcError extends Error {
  code: number;
  data?: unknown;
}
```
* message
	* Must be a human-readable string
	* Should adhere to the specifications in the [Error Standards](https://eips.ethereum.org/EIPS/eip-1193#error-standards)
* code
	* Must be an integer number
	* Should adhere to the specifications in the [Error Standards](https://eips.ethereum.org/EIPS/eip-1193#error-standards)
* data
	* Should contain any other useful information about the error

### [Error Standards]([Error Standards](https://eips.ethereum.org/EIPS/eip-1193#error-standards))

### Events
The Provider must implement the following event handling methods:
* `on`
* `removeListener`
	* These methods must be implemented per the Node.js `EventEmitter` API.

The `message` event is intended for arbitrary notifications not covered by other events. When emitted, the message event must be emitted with an object argument of the following form:
```
interface ProviderMessage {
  readonly type: string;
  readonly data: unknown;
}
```

### Subscriptions
If the Provider supports Ethereum RPC subscriptions, the Provider must emit the `message` event when it receives a subscription notification. If the Provider receives a subscription message from e.g. an `eth_subscribe subscription`, the Provider must emit a `message` event with a `ProviderMessage` object of the following form:
```
interface EthSubscription extends ProviderMessage {
  readonly type: 'eth_subscription';
  readonly data: {
    readonly subscription: string;
    readonly result: unknown;
  };
}
```
 
### connect
If the Provider becomes connected, the Provider must emit the event named `connect`. This includes when:
* The Provider first connects to a chain after initialization.
* The Provider connects to a chain after the disconnect event was emitted.
This event must be emitted with an object of the following form:
```
interface ProviderConnectInfo {
  readonly chainId: string;
}
```
`chainId` must specify the integer ID of the connected chain as a hexadecimal string, per the `eth_chainId` Ethereum RPC method.

### disconnect
If the Provider becomes disconnected from all chains, the Provider must emit the event named `disconnect` with `value error: ProviderRpcError`, per the interfaced defined in the RPC Errors section. The value of the error’s `code` property must follow the status codes for CloseEvent.

### chainChanged
If the chain the Provider is connected to changes, the Provider must emit the event named `chainChanged` with value `chainId: string`, specifying the integer ID of the new chain as a hexadecimal string, per the `eth_chainId` Ethereum RPC method.

### accountsChanged
If the accounts available to the Provider change, the Provider must emit the event named `accountsChanged` with `value accounts: string[]`, containing the account addresses per the `eth_accounts` Ethereum RPC method. The “accounts available to the Provider” change when the return value of `eth_accounts` changes.