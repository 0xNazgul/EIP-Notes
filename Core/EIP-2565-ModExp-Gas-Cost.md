# EIP-2565 ModExp Gas Cost
Defines the gas cost of the `ModExp 0x00..05` precompile and specifies an algorithm for calculating the gas cost. It approximates the multiplication complexity cost and multiplies that by an approximation of iterations to execute.

## Motivation
Modular exponentiation si a foundational arithmetic operation for cryptographic functions (signature, VDFs, SNARKs, accumulators, etc). The `ModExp` is currently over priced and making these inefficient/expensive.

## Specification
The gas cost of calling the precompile at address `0x00..05` will be calculated as follows:
```
def calculate_multiplication_complexity(base_length, modulus_length):
    max_length = max(base_length, modulus_length)
    words = math.ceil(max_length / 8)
    return words**2

def calculate_iteration_count(exponent_length, exponent):
    iteration_count = 0
    if exponent_length <= 32 and exponent == 0: iteration_count = 0
    elif exponent_length <= 32: iteration_count = exponent.bit_length() - 1
    elif exponent_length > 32: iteration_count = (8 * (exponent_length - 32)) + ((exponent & (2**256 - 1)).bit_length() - 1)
    return max(iteration_count, 1)

def calculate_gas_cost(base_length, modulus_length, exponent_length, exponent):
    multiplication_complexity = calculate_multiplication_complexity(base_length, modulus_length)
    iteration_count = calculate_iteration_count(exponent_length, exponent)
    return max(200, math.floor(multiplication_complexity * iteration_count / 3))
```

## Rationale
With benchmarking it was discovered that `ModExp` was overpriced and it's current formula could be improved for better estimates. The changes are as follows:
1. Modify ‘computational complexity’ formula to better reflect the computational complexity
2. Change the value of `GQUADDIVISOR`
3. Set a minimum gas cost to prevent abuse
