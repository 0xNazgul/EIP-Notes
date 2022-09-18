# EIP-158 State clearing

## Specification
For all blocks where `block.number >= FORK_BLKNUM`:
1. Where a state change is made to an account and this state change results in the account state being saved with nonce = 0, balance = 0, code empty, storage empty (an "empty account"), the account is deleted
2. If a address is touched and that address contains an empty account, then it is deleted. A touch is as any situation where if the account at the given address were nonexistent it would be created. (SEE NOTE 2)
3. Whenever the EVM checks if an account exists, emptiness is treated as equivalent to nonexistence. 
* This implies that there is no longer a meaningful difference between emptiness and nonexistence from the point of view of EVM execution.
4. Zero-value calls and zero-value suicides no longer consume the 25,000 account creation gas cost in any circumstance.

Where a touch takes place:
* Zero-value-bearing `CALL`s
* `CREATE`s (when the code that is ultimately saved is empty and there is no ether remaining in the account when it is saved)
* Zero-value-bearing `SUICIDE`s
* TX recipients
* Contracts created in contract creation TXs
* Miners receiving TX fees (NOTE the case where the gas price is 0, and the account does not yet exist because it only receives the block/uncle/nephew rewards after processing every TX)

Emptiness is defined by `is_empty(acct): return get_balance(acct) == 0 and get_code(acct) == "" and get_nonce(acct) == 0`.
* Emptiness of storage does not matter, simplifying client implementation 

NOTE 2: Don't implement this ie. no new empty accounts can be created, but existing ones are not automatically destroyed unless their state is actually changed. instead during each block starting from N and ending when there are no null accounts left, select the 1000 null accounts that are left-most in order of sha3(address) and delete them.

## Rationale
This will remove empty accounts that have been put in the state at a low cost due to flaws. Reducing state size, hard disk load of a full client and reducing time for fast sync.
* Simplifies the protocol for there is no longer a distinction between an empty account and a nonexistent one. 

This introduces a temporary breaking of existing guarantees in that by repeatedly zero-value-calling already existing empty accounts one can create a state change at a cost of `700` gas per account instead of `5000` per gas minimum (`SUICIDE` refunds this to do 350 gas per account). 
* This will increase state writes per block and therefore heighten block processing times and increase uncle rates short term during the clearing. 