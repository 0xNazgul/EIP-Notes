# EIP-198 Big integer modular exponentiation

## Specification
Address `0x00……05, has a precompile that takes input in the following format:
```js
<length_of_BASE> <length_of_EXPONENT> <length_of_MODULUS> <BASE> <EXPONENT> <MODULUS>
```
Every length is a 32-byte left-padded integer representing the number of bytes to be taken up by the next value. Call data is assumed to be infinitely right-padded with zero bytes (excess data is ignored).
* Consumes `floor(mult_complexity(max(length_of_MODULUS, length_of_BASE)) * max(ADJUSTED_EXPONENT_LENGTH, 1) / GQUADDIVISOR)` gas. 
* If there is enough gas, returns an output `(BASE**EXPONENT) % MODULUS` as a byte array with the same length as the modulus

`ADJUSTED_EXPONENT_LENGTH` is as:
* If `length_of_EXPONENT <= 32`, all bits in `EXPONENT` are 0, return 0
* If `length_of_EXPONENT <= 32`, return the index of the highest bit in `EXPONENT`.
* If length_of_EXPONENT > 32, return `8 * (length_of_EXPONENT - 32)` with the index of the highest bit in the first 32 bytes of `EXPONENT` (if `EXPONENT = \x00\x00\x01\x00.....\x00`, with one hundred bytes, the result is `8 * (100 - 32) + 253 = 797`). If all of the first 32 bytes of `EXPONENT` are 0, return exactly `8 * (length_of_EXPONENT - 32)`.

`mult_complexity` is a function intended to approximate the difficulty of [Karatsuba multiplication](https://en.wikipedia.org/wiki/Karatsuba_algorithm) (used in bigint libraries) and is as follows:
```js
def mult_complexity(x):
    if x <= 64: return x ** 2
    elif x <= 1024: return x ** 2 // 4 + 96 * x - 3072
    else: return x ** 2 // 16 + 480 * x - 199680
```

## Example
Input data:
```j
0000000000000000000000000000000000000000000000000000000000000001
0000000000000000000000000000000000000000000000000000000000000020
0000000000000000000000000000000000000000000000000000000000000020
03
fffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2e
fffffffffffffffffffffffffffffffffffffffffffffffffffffffefffffc2f
```
This represents the exponent `3**(2**256 - 2**32 - 978) % (2**256 - 2**32 - 977)` which equals 1 ([Fermat's little theorem](https://en.wikipedia.org/wiki/Fermat%27s_little_theorem)):
`0000000000000000000000000000000000000000000000000000000000000001`

* It's returned as 32 bytes since the modulus length was 32 bytes. `ADJUSTED_EXPONENT_LENGTH` would be `255` and the gas cost would be `mult_complexity(32) * 255 / 20 = 13056`. A 4096-bit [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) would cost `mult_complexity(512) * 4095 / 100 = 22853376`.
* Most cases RSA verification uses an exponent of 3 or 65537(reduces consumption to `5580 or 89292`)

## Rationale
This allows efficient RSA verification in the EVM as well as other number theory-based cryptography. 