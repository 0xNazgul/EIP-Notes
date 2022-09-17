# EIP-150: Gas cost changes for IO-heavy operations

## Specification
* Increase gas cost of `EXTCODESIZE` from `20 -> 700`
* increase base gas cost of `EXTCODECOPY` from `20->700`
* Increase the gas cost of `BALANCE` FROM `20 -> 400`
* Increase the gas cost of `CALL, DELEGATECALL, CALLCODE` from `40->700`
* Increase the gas cost of `SELFDESTRUCT` from `0 -> 5_000`
* If `SELFDESTRUCT` hits a newly created account it the gas cost is an additional `25_000`
* Recommended gas limit target is 5.5 million
* Define “all but one 64th” of `N` as `N - floor(N / 64)`
* If a call asks for more gas than max allowed amount (doesn't return OOG error).
* If a call asks for more gas than all but one 64th of the max allowed amount, call with all but one 64th of the maximum allowed amount of gas.
* `CREATE` only provides all but one 64th of the parent gas to the child call.

## Rationale
DoS attacks proved that opcodes that read the stae tree are under-priced and can be used to degrade network performance via TX spam. The concern:
* It takes a long time to read from disk and a risk to future sharding proposals as the attack TX that force the network to require tens of megabytes to provide Merkle proofs for.

This attempts to target a limit of 8mb of data that needs to be read in order to process a block. Also, include an estimate of 500 bytes for a merkle proof for `SLOAD` and `1_000` for an account.

It introduces [EIP-90](https://github.com/ethereum/EIPs/issues/90) for without it all current contracts that make calls would stop working for they use `msg.gas - 40` to see how much gas to make a call with. 

Also introduces [EIP-114](https://github.com/ethereum/EIPs/issues/114) that achieves the benefit of replacing the call stack depth limit with a soft gas-based restriction. Eliminating call stack dpth attacks for contract developers. 

The de-facto maximum call stack depth is limited from  `~1024 -> ~340` mitigating the harm from potential quadratic-complexity DoS attacks that rely on calls. Also the gas limit increase is recommended (Done in EIP-1559).