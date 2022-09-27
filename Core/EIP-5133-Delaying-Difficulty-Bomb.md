# EIP-5133: Delaying Difficulty Bomb to mid-September 2022
(All the same reasoning as before)

## Specification
For the purposes of `calc_difficulty`, simply replace the use of `block.number`, as used in the exponential ice age component, with the formula:
`fake_block_number = max(0, block.number - 11_400_000) if block.number >= FORK_BLOCK_NUMBER else block.number`