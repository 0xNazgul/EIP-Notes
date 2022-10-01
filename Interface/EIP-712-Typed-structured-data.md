# EIP-712: Typed structured data hashing and signing
This is a standard for hashing and signing of typed structured data as opposed to just bytestrings (doesn't include replay protection). 

## Specification
### Signatures and Hashing overview
* Signature scheme consists of hashing algorithm  (`keccak256`) and a signing algorithm (`secp256k1`).

### Transactions and bytestrings
Ethereum has two kinds of messages, TX `𝕋` and bytestrings `𝔹⁸ⁿ`. They are signed using `eth_sendTransaction` and `eth_sign`. Encoding function `encode : 𝕋 ∪ 𝔹⁸ⁿ → 𝔹⁸ⁿ` was defined as:
* `encode(t : 𝕋) = RLP_encode(t)`
* `encode(b : 𝔹⁸ⁿ) = b`
	* Individually they satisfy the required properties but not together. With `b = RLP_encode(t)` there is a collision. So, the modification of the second leg:
* `encode(b : 𝔹⁸ⁿ) = "\x19Ethereum Signed Message:\n" ‖ len(b) ‖ b` where `len(b)` is the ascii-decimal encoding of the number of bytes in `b`.
	* Solves the collision between the legs since `RLP_encode(t : 𝕋)` never starts with `\x19`. There is still the risk of the new encoding function not being deterministic or injective.
This is not deterministic, for a 4-byte string `b` both encodings with `len(b) = "4"` and `len(b) = "004"` are valid. Soved with requiring that the decimal encoding of the length has no leading zeros and `len("") = "0".


Both determinism and injectiveness would be trivially true if `len(b)` was left out entirely. It is difficult to map arbitrary sets to bytestrings without additional security issues in the encoding function. 

### Arbitrary messages
`eth_sign` call assumes messages to be bytestrings. It is not hashing bytestrings but the collection of all semantically different messages of all different DApps `𝕄`. This set is impossible to formalize, so instead approximate it with the set of typed named structures `𝕊`. This Formalizes the set `𝕊` and provides a deterministic injective encoding function for it.
* Just encoding structs is not enough because it's likely that two different DApps use identical structs. When this happens, a signed message intended for one DApp would also be valid for the other. The signatures are compatible and it can be intended behaviour, in such it's fine as long as the DApps took replay attacks into consideration. 
The way to solve this is by introducing a domain separator, a 256-bit number. This is a value unique to each domain that is ‘mixed in’ the signature. It makes signatures from different domains incompatible. The domain separator is designed to include bits of DApp unique information such as the name of the DApp, the intended validator contract address, the expected DApp domain name, etc. The user and user-agent can use this information to mitigate phishing attacks, where a malicious DApp tries to trick the user into signing a message for another DApp.

## Specification pt2
The set of signable messages is extended from transactions and bytestrings `𝕋 ∪ 𝔹⁸ⁿ` to also include structured data `𝕊`. The new set of signable messages is thus `𝕋 ∪ 𝔹⁸ⁿ ∪ 𝕊`. Are encoded to bytestrings suitable for hashing and signing as follows:
* `encode(transaction : 𝕋) = RLP_encode(transaction)`
* `encode(message : 𝔹⁸ⁿ) = "\x19Ethereum Signed Message:\n" ‖ len(message) ‖ message` where `len(message)` is the non-zero-padded ascii-decimal encoding of the number of bytes in `message`.
* `encode(domainSeparator : 𝔹²⁵⁶, message : 𝕊) = "\x19\x01" ‖ domainSeparator ‖ hashStruct(message)` where `domainSeparator` and `hashStruct(message)` are defined below.
It's deterministic since individual components are. The encoding is injective since the 3 cases always differ in first byte (`RLP_encode(transaction)` does not start with `\x19`). 

### Definition of typed structured data `𝕊`
To define the set of all structured data, we start with defining acceptable types. Like `ABIv2` these are closely related to Solidity types. Example:
```
struct Mail {
    address from;
    address to;
    string contents;
}
```
* `struct type` has valid identifier as name and contains zero or more member variables. Member variables have a member type and a name.
* `member type` can be either an atomic type, a dynamic type or a reference type.
* `atomic types` are `bytes1 to bytes32, uint8 to uint256, int8 to int256, bool and address`. 
	* There are no aliases `uint and int`. 
	* Contract addresses are always plain `address`. Fixed point numbers are not supported by the standard. Future versions of this standard may add new atomic types.
* `dynamic types` are bytes and string. These are like the atomic types for the purposed of type declaration, but their treatment in encoding is different.
* `reference types` are arrays and structs. Arrays are either fixed size or dynamic and denoted by `Type[n] or Type[]` respectively. Structs are references to other structs by their name. The standard supports recursive struct types.
* The set of structured typed data `𝕊` contains all the instances of all the struct types.

### Definition of `hashStruct`
Function is:
* `hashStruct(s : 𝕊) = keccak256(typeHash ‖ encodeData(s)) where typeHash = keccak256(encodeType(typeOf(s)))`
	* The `typeHash` is a constant for a given struct type and does not need to be runtime computed.

### Definition of `encodeType`
The type of a struct is encoded as `name ‖ "(" ‖ member₁ ‖ "," ‖ member₂ ‖ "," ‖ … ‖ memberₙ ")"` 
* Each member is written as `type ‖ " " ‖ name`. 
* If the struct type references other struct types, then the set of referenced struct types is collected, sorted by name and appended to the encoding.

### Definition of encodeData
Encoding of a struct instance is `enc(value₁) ‖ enc(value₂) ‖ … ‖ enc(valueₙ), ...` the concatenation of the encoded member values in the order that they appear in the type. Each encoded member value is exactly 32-byte long.
* Atomic values are encoded as follows: 
	* Boolean `false and true` are encoded as `uint256` values 0 and 1. 
	* Addresses are encoded as `uint160`. 
	* Integer values are sign-extended to 256-bit and encoded in big endian order. 
	* `bytes1 to bytes31` are arrays with a beginning (index 0) and an end (index length - 1), they are zero-padded at the end to `bytes32` and encoded in beginning to end order. This corresponds to their encoding in `ABI v1 and v2`.
* Dynamic values `bytes and string` are encoded as a `keccak256` hash of their contents.
* Array values are encoded as the `keccak256` hash of the concatenated `encodeData` of their contents.
* Struct values are encoded recursively as `hashStruct(value)` and this is undefined for cyclical data.

### Definition of domainSeparator
```
domainSeparator = hashStruct(eip712Domain)
```
The type of `eip712Domain` is a struct named `EIP712Domain` with one or more of the below fields. Protocol designers only need to include the fields that make sense for their signing domain. Unused fields are left out of the struct type.
* `string name` the user readable name of signing domain, i.e. the name of the DApp or the protocol.
* `string version` the current major version of the signing domain. Signatures from different versions are not compatible.
* `uint256 chainId` the EIP-155 chain id. The user-agent should refuse signing if it does not match the currently active chain.
* `address verifyingContract` the address of the contract that will verify the signature. The user-agent may do contract specific phishing prevention.
* `bytes32 salt` an disambiguating salt for the protocol. This can be used as a domain separator of last resort.

### Specification of the eth_signTypedData JSON RPC
The method `eth_signTypedData` is added to the Ethereum JSON-RPC.
* Calculates an Ethereum specific signature with: `sign(keccak256("\x19Ethereum Signed Message:\n" + len(message) + message)))`
	* Adding a prefix to the message makes the calculated signature recognizable as an Ethereum specific signature. This prevents misuse where a malicious DApp can sign arbitrary data and use the signature to impersonate the victim.
Parameters
1. `Address`: 20 Bytes - Address of the account that will sign the messages.
2. `TypedData`: Typed structured data to be signed.
Typed data is a JSON object containing type information, domain separator parameters and the message object. Here is the json-schema definition for `TypedData` param.
```
{
  type: 'object',
  properties: {
    types: {
      type: 'object',
      properties: {
        EIP712Domain: {type: 'array'},
      },
      additionalProperties: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            name: {type: 'string'},
            type: {type: 'string'}
          },
          required: ['name', 'type']
        }
      },
      required: ['EIP712Domain']
    },
    primaryType: {type: 'string'},
    domain: {type: 'object'},
    message: {type: 'object'}
  },
  required: ['types', 'primaryType', 'domain', 'message']
}
```
Returns
`DATA`: Signature. As in `eth_sign` it is a hex encoded 129 byte array starting with `0x`. It encodes the `r, s and v` parameters from appendix F of the yellow paper in big-endian format. Bytes 0…64 contain the `r` parameter, bytes 64…128 the `s` parameter and the last byte the `v` parameter(v includes chain id).
* There also should be a corresponding `personal_signTypedData` method which accepts the password for an account as the last argument.

### Specification of the Web3 API
Two methods are added to Web3.js version 1 that parallel the `web3.eth.sign and web3.eth.personal.sign` methods.

web3.eth.signTypedData
```
web3.eth.signTypedData(typedData, address [, callback])
```
* Signs typed data using a specific account.

Parameters
1. `Object`: Domain separator and typed data to sign. Structured according to the JSON-Schema specified above in the `eth_signTypedData` JSON RPC call.
2. `String|Number`: Address to sign data with. Or an address or index of a local wallet in `:ref:web3.eth.accounts.wallet <eth_accounts_wallet>`.
* `String|Number` address parameter can also be an address or index from the `web3.eth.accounts.wallet <eth_accounts_wallet>`. It will then sign locally using the private key of this account.
3. `Function`: (optional) Optional callback, returns an error object as first parameter and the result as second.
Returns
`Promise` returns `String`: The signature as returned by `eth_signTypedData`.