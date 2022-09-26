# EIP-3529 Reduction in refunds
Remove gas refunds for `SELFDESTRUCT`, and reduce gas refunds for `SSTORE` where refunds are still good, but no longer high enough for current exploits of the refund mechanism.

## Motivation
Gas refunds for `SSTORE and SELFDESTRUCT` were introduced to motivate app devs to write app that practice clearing storage slots and contracts that are no longer needed. However, the benefits of this technique have proven to be far lower than anticipated, and gas refunds have had multiple unexpected harmful consequences:
1. Refunds give rise to GasToken. GasToken has benefits in moving gas space from low-fee periods to high-fee periods, but it also has downsides to the network. Particularly in exacerbating state size and inefficiently clogging blockchain gas usage
2. Refunds increase block size variance. The theoretical maximum amount of actual gas consumed in a block is nearly twice the on-paper gas limit. This is not fatal, but is still undesirable, given that refunds can be used to maintain 2x usage spikes for far longer than EIP-1559 can.

## Specification
`FORK_BLOCK`:TBD
`MAX_REFUND_QUOTIENT`:5
The following changes apply:
* Remove `SELFDESTRUCT` refund.
* Replace `SSTORE_CLEARS_SCHEDULE` with `SSTORE_RESET_GAS + ACCESS_LIST_STORAGE_KEY_COST`
* Reduce the max gas refunded after a transaction to `gas_used // MAX_REFUND_QUOTIENT`
Remark: Previously max gas refunded was defined as `gas_used // 2`. Here we name the constant 2 as `MAX_REFUND_QUOTIENT` and change its value to 5.
