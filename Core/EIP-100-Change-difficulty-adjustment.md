# EIP-100: Change difficulty adjustment to target mean block time including uncles

## Specification
Current formula:
```
adj_factor = max(1 - ((timestamp - parent.timestamp) // 10), -99)
child_diff = int(max(parent.difficulty + (parent.difficulty // BLOCK_DIFF_FACTOR) * adj_factor, min(parent.difficulty, MIN_DIFF)))
```

IF `block.number >= BYZANTIUM_FORK_BLKNUM` formula is:
```
adj_factor = max((2 if len(parent.uncles) else 1) - ((timestamp - parent.timestamp) // 9), -99)
```

## Rationale
The change to the formula is to ensure that difficulty adjustment algorithm targets a constant average rate of blocks produced including uncles. Ensures a predictable issuance rate preventing manipulation upward by manipulating the uncle rate.

This formula accounts for the exact number of included uncles:
```
adj_factor = max(1 + len(parent.uncles) - ((timestamp - parent.timestamp) // 9), -99)
```
This was chosen for it is mathematically equivalent to assuming that a block with `k` uncles is equivalent to a sequence of `k+1` blocks that all appear with the same timestamp. The formula depends on the full block (not just header) so is an approximate formula to get the same effect (has the benefit of it only depends on the block header).
* You can check the uncle hash against the blank hash.
* Change of the denominator from `10->9` ensures the block time remains the same.
