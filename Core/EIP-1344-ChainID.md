# EIP-1344 ChainID
`CHAINID 0x46` will return the current chains unique identifier. It uses 0 stack arguments and pushes the current chainID onto the stack. 
* 256-bit value and cost `G_base` to execute
* Will help against replay attacks on signed messages.
Since there is no spec for how chainID is set for a network it relies on choices made manually by the client and chain community. This produces a case during a `contentious split` over a divisive issue. A community using a value of chainID will make a decision to split into two such chains
* It will be unsafe to maintain chainID to the same value on both chains.
* Two solutions to this:
1. one chain modifies their value of chainID (other keeps it)
2. both chains modify their value of chainID

To mitigate this user of `CHAINID` must ensure that their application can handle it if this occurs. 
