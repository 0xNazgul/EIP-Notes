# EIP-141 Designated invalid EVM instruction

# Specification
`INVALID 0xfe` is used to abort execution. Equivalent to REVERT (since Byzantium fork) with 0,0 as stack parameters, except that all the gas given to the current context is consumed.
