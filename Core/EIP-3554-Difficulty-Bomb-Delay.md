# EIP-3554: Difficulty Bomb Delay to December 2021
(All the same reasoning as before)

## Specification
For the purposes of `calc_difficulty`, simply replace the use of `block.number`, as used in the exponential ice age component, with the formula:
`fake_block_number = max(0, block.number - 9_700_000) if block.number >= FORK_BLOCK_NUMBER else block.number`
