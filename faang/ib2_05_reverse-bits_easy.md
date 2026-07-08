## challenge: Reverse Bits
tags: bit-tricks, divide-and-conquer
track: faang
difficulty: easy

Reverse the bits of a given 32-bit unsigned integer `n`. That is, the least significant bit of the input becomes the most significant bit of the output, and vice versa. Note that the input is treated as an unsigned value, so bit 0 maps to bit 31, bit 1 maps to bit 30, and so on.

Constraints: the input is a 32-bit unsigned integer (`0 <= n <= 2^32 - 1`).

Example: `n = 43261596` (binary `00000010100101000001111010011100`) → `964176192` (binary `00111001011110000010100101000000`). Example: `n = 4294967293` → `3221225471`.

hint: Build the answer bit by bit: shift the result left, then append the lowest bit of `n`.
hint: Over 32 iterations, `result = (result << 1) | (n & 1)` followed by `n >>= 1` places input bit `i` at output position `31 - i`.
hint: Use unsigned arithmetic throughout so the right shift on `n` fills with zeros.

```cpp
// starter
#include <cstdint>
uint32_t reverseBits(uint32_t n);
```

```cpp
uint32_t reverseBits(uint32_t n) {
    uint32_t result = 0;
    for (int i = 0; i < 32; ++i) {
        result = (result << 1) | (n & 1u);
        n >>= 1;
    }
    return result;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    if (reverseBits(43261596u) != 964176192u)     { std::puts("case1"); return 1; }
    if (reverseBits(4294967293u) != 3221225471u)  { std::puts("case2"); return 1; }
    if (reverseBits(0u) != 0u)                     { std::puts("case3"); return 1; }
    if (reverseBits(1u) != 2147483648u)            { std::puts("case4"); return 1; }
    if (reverseBits(4294967295u) != 4294967295u)   { std::puts("case5"); return 1; }
    if (reverseBits(2147483648u) != 1u)            { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Process all 32 bit positions once. Each iteration shifts the accumulated result one place left to make room, ORs in the current least significant bit of `n`, then discards that bit with a right shift. After 32 steps the bit that started at position `i` lands at position `31 - i`, exactly the reversal. Using `uint32_t` keeps the shifts logical (zero-filled). O(1) time (a fixed 32 iterations) and O(1) space.
