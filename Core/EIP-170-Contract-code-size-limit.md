# EIP-170 Contract code size limit
If `block.number >= FORK_BLKNUM` then if contract creation initialization returns data with length of more than `MAX_CODE_SIZE` bytes it fails with out of gas error.

# Rationale
Current quadratic vulnerability in Ethereum:
* When a contract is called even though the call takes a constant amount of gas the call can trigger O(n) cost in terms of reading the code from disk, preprocessing the code for VM execution, and adding O(n) data to the Merkle proof for the block's proof-of-validity. 
* At current gas levels this is acceptable but at higher gas levels this would create issues.
* Simple solution of putting a hard cap on the size of an object that can be saved to the blockchain and do so that it is feasible with current gas limits.
