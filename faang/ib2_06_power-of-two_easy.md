## challenge: Power of Two
tags: bit-tricks, math
track: faang
difficulty: easy

Given an integer `n`, return `true` if it is a power of two, otherwise return `false`. An integer `n` is a power of two if there exists an integer `x` such that `n == 2^x`. Note that `2^0 == 1` counts, and zero and negative numbers are never powers of two.

Constraints: `-2^31 <= n <= 2^31 - 1`.

Example: `n = 1` → `true` (`2^0`). Example: `n = 16` → `true` (`2^4`). Example: `n = 3` → `false`.

hint: A positive power of two has exactly one bit set in its binary representation.
hint: For a one-bit number `n`, the expression `n & (n - 1)` clears that single bit, yielding 0.
hint: Guard against zero and negatives first, since `n & (n - 1) == 0` is also true for `n == 0`.

```cpp
// starter
bool isPowerOfTwo(int n);
```

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (isPowerOfTwo(1) != true)           { std::puts("case1"); return 1; }
    if (isPowerOfTwo(16) != true)          { std::puts("case2"); return 1; }
    if (isPowerOfTwo(3) != false)          { std::puts("case3"); return 1; }
    if (isPowerOfTwo(0) != false)          { std::puts("case4"); return 1; }
    if (isPowerOfTwo(-16) != false)        { std::puts("case5"); return 1; }
    if (isPowerOfTwo(5) != false)          { std::puts("case6"); return 1; }
    if (isPowerOfTwo(1073741824) != true)  { std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** In binary a power of two is a single set bit, e.g. `1, 10, 100, ...`. Subtracting 1 from such a number flips that bit to 0 and turns every lower bit to 1, so `n & (n - 1)` erases the only set bit and gives 0. The check `n > 0` rules out zero (which also satisfies the AND test) and all negatives (whose sign bit is set). The whole test is O(1) time and O(1) space.
