# EIP-2384: Muir Glacier Difficulty Bomb Delay
(all the same information as before) This time it'll delay the ice Age for ~611 days.

For the purposes of `calc_difficulty`, simply replace the use of `block.number`, as used in the exponential ice age component, with the formula:
`fake_block_number = max(0, block.number - 9_000_000) if block.number >= MUIR_GLACIER_FORK_BLKNUM else block.number`
