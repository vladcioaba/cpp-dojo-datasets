## challenge: popcount without the builtin
tags: bit-tricks
track: hft

Count set bits. Kernighan's trick clears the lowest set bit each iteration (`x &= x - 1`), so it loops once per set bit rather than 64 times. Implement `int popcount(uint64_t x)` without `__builtin_popcount` / `std::popcount`.

hint: You do not need to inspect all 64 bits — only the set ones matter.
hint: `x &= x - 1` clears the lowest set bit; count how many times you can do it before x hits zero.

```cpp
int popcount(uint64_t x) {
    int n = 0;
    while (x) { x &= x - 1; ++n; }
    return n;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    if (popcount(0) != 0) { std::puts("0 wrong"); return 1; }
    if (popcount(1) != 1) { std::puts("1 wrong"); return 1; }
    if (popcount(0xFFull) != 8) { std::puts("0xFF wrong"); return 1; }
    if (popcount(~0ull) != 64) { std::puts("all-ones wrong"); return 1; }
    if (popcount(0xF0F0ull) != 8) { std::puts("0xF0F0 wrong"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Kernighan's method: subtracting 1 flips the lowest set bit and all zeros beneath it, so `x & (x - 1)` clears exactly that one bit. The loop therefore runs once per set bit rather than a full 64 times, giving O(popcount) iterations and O(1) space.
