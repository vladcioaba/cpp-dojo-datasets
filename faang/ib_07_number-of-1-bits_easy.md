## challenge: Number of 1 Bits
tags: bit-tricks, divide-and-conquer
track: faang
difficulty: easy

Write a function that takes an unsigned 32-bit integer `n` and returns the number of `1` bits it has (also known as its Hamming weight).

Constraints: the input is a 32-bit unsigned integer, `0 <= n <= 2^32 - 1`.

Example: `n = 11` (binary `1011`) → `3`. Example: `n = 128` (binary `10000000`) → `1`. Example: `n = 4294967293` (binary `11111111111111111111111111111101`) → `31`.

hint: The naive way tests all 32 bit positions; there is a trick that loops only once per set bit.
hint: The expression `n & (n - 1)` clears the lowest set bit of `n` and leaves the rest untouched.
hint: Repeatedly apply `n &= (n - 1)`, counting iterations, until `n` becomes 0 — that count is the number of one bits (Brian Kernighan's algorithm).

```cpp
// starter
#include <cstdint>
int hammingWeight(uint32_t n);
```

```cpp
int hammingWeight(uint32_t n) {
    int count = 0;
    while (n) {
        n &= (n - 1);
        ++count;
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    if (hammingWeight(11u) != 3)          { std::puts("case1"); return 1; }
    if (hammingWeight(128u) != 1)         { std::puts("case2"); return 1; }
    if (hammingWeight(4294967293u) != 31) { std::puts("case3"); return 1; }
    if (hammingWeight(0u) != 0)           { std::puts("case4"); return 1; }
    if (hammingWeight(4294967295u) != 32) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Subtracting 1 flips the lowest set bit to 0 and turns every bit below it into 1; ANDing that with the original clears exactly that lowest set bit. So `n &= (n - 1)` removes one set bit per iteration, and the loop runs once for each `1` in the number rather than once for each of the 32 positions. Time is O(number of set bits), space O(1).
