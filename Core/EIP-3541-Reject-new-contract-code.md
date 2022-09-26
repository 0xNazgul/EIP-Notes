# EIP-3541: Reject new contract code starting with the `0xEF` byte 
Disallow new code starting with the `0xEF` byte to be deployed. Code already existing in the account trie starting with `0xEF` byte is not affected.

## Motivation
Contracts conforming to the `EVM Object Format (EOF)` are going to be validated at deploy time. To guarantee that every `EOF-formatted` contract in the state is valid, it is needed to prevent already deployed contracts from being recognized as such format. Achieved by choosing a byte sequence for the magic that doesn’t exist in any of the already deployed contracts. To prevent the growth of the search space and to limit the analysis to the contracts existing before this fork, we disallow the starting byte of the format (the first byte of the magic).
* Should the `EVM Object Format` proposal not be deployed, the magic can be used by other features depending on versioning. In the case versioning becomes obsolete, it is simple to roll this back by allowing contracts starting with the `0xEF` byte to be deployed again
