## challenge: Sum of Two Integers
tags: bit-tricks, math
track: faang
difficulty: hard

Given two integers `a` and `b`, return their sum without using the operators `+` or `-`. You may use only bitwise operations, loops, and comparisons.

Constraints: `-1000 <= a, b <= 1000`.

Example: `a = 1, b = 2` → `3`. Example: `a = 2, b = 3` → `5`. Example: `a = -2, b = 3` → `1`.

hint: Binary addition splits into two independent pieces: the sum ignoring carries, and the carries themselves.
hint: The carry-free sum of two bits is their XOR (`a ^ b`); the positions that generate a carry are `a & b`, and each carry lands one position to the left (`<< 1`).
hint: Loop: replace `a` with `a ^ b` and `b` with `(a & b) << 1` until there is no carry left (`b == 0`). Do the shift on an `unsigned` value so negative numbers behave correctly.

```cpp
// starter
int getSum(int a, int b);
```

```cpp
int getSum(int a, int b) {
    while (b != 0) {
        unsigned carry = (unsigned)(a & b) << 1;
        a = a ^ b;
        b = (int)carry;
    }
    return a;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (getSum(1, 2)   != 3)   { std::puts("case1"); return 1; }
    if (getSum(2, 3)   != 5)   { std::puts("case2"); return 1; }
    if (getSum(-2, 3)  != 1)   { std::puts("case3"); return 1; }
    if (getSum(0, 0)   != 0)   { std::puts("case4"); return 1; }
    if (getSum(-5, -7) != -12) { std::puts("case5"); return 1; }
    if (getSum(1000, -1) != 999) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Full-adder logic expressed on whole words. `a ^ b` adds the two numbers as if no carries occurred, while `a & b` marks every bit position that produces a carry; shifting that left by one moves each carry into the column where it must be added next. Iterating "sum without carry, then fold the carry back in" terminates once the carry word is 0. Computing the carry through an `unsigned` avoids undefined behavior from shifting or overflowing a signed value, and C++20's guaranteed two's-complement representation makes the final `unsigned`-to-`int` conversion reproduce the correct negative result. O(1) iterations (bounded by the word width), O(1) space.
