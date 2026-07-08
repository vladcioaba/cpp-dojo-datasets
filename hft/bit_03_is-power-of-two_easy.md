## challenge: Is it a power of two
tags: bit-tricks, hft
track: hft
difficulty: easy

Given a 32-bit unsigned integer, return `true` if it is a power of two (exactly one bit set) and `false` otherwise. Zero is not a power of two. Implement `bool isPowerOfTwo(uint32_t x)`. Sizing ring buffers and hash tables to powers of two turns modulo into a mask, so this check guards those invariants.

Constraints: `0 <= x <= 2^32 - 1`. O(1), no loop.

Example: `isPowerOfTwo(16) == true`. Example: `isPowerOfTwo(0) == false`, `isPowerOfTwo(3) == false`.

hint: A power of two has a single set bit; clearing the lowest set bit of such a number leaves zero.
hint: `x & (x - 1)` clears the lowest set bit — for a power of two that empties the whole word.
hint: Also exclude `0`, which would otherwise pass the mask test. Return `x != 0 && (x & (x - 1)) == 0`.

```cpp
// starter
#include <cstdint>
bool isPowerOfTwo(uint32_t x);
```

```cpp
bool isPowerOfTwo(uint32_t x) {
    return x != 0u && (x & (x - 1u)) == 0u;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { uint32_t x; bool want; } cases[] = {
        {0u, false},           // zero is not a power of two
        {1u, true},            // 2^0
        {2u, true},            // 2^1
        {3u, false},           // two bits set
        {16u, true},           // 2^4
        {0x80000000u, true},   // 2^31 (INT_MIN pattern)
        {0xFFFFFFFFu, false},  // all ones
    };
    for (auto& c : cases) {
        bool got = isPowerOfTwo(c.x);
        if (got != c.want) { std::printf("isPowerOfTwo(%u)=%d want %d\n", c.x, (int)got, (int)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** A power of two has exactly one set bit. Clearing the lowest set bit with `x & (x - 1)` removes that single bit and yields 0 iff there was only one bit to begin with. The mask test alone would also accept `x == 0` (whose `x & (x-1)` is `0 & 0xFFFFFFFF == 0`), so the explicit `x != 0` guard excludes it. Constant-time, branchless arithmetic; the `&&` short-circuit is a cheap predicate, not a data-dependent loop.
