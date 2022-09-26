# EIP-3198 BASEFEE opcode
Adds an new opcode that gives the EVM access to the block’s base fee.
* `BASEFEE 0x48` returns the value of the base fee of the current block it is executing in.

## Motivation
Use case is to be for contracts to get the value of the base fee improving:
1. Contracts that need to set bounties for anyone to poke them with a TX could set the bounty to be `BASEFEE + x, or BASEFEE * (1 + x)`. This makes the mechanism more reliable for they will always pay “enough” regardless of market conditions.
2. Gas futures can be implemented based on this and would be more precise than gastokens.
3. Improved security for state channels, plasma, optirolls and other fraud proof driven solutions. Having the `BASEFEE` as an input allows to lengthen the challenge period automatically if you see that the `BASEFEE` is high.
