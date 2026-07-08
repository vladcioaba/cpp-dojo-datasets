## challenge: Swap two ints without a temporary (XOR)
tags: bit-tricks, hft
track: hft
difficulty: easy

Swap the values of two `int`s in place using only XOR — no temporary variable and no addition. Implement `void xorSwap(int& a, int& b)`. It is an interview staple for testing whether you know the aliasing gotcha; in practice `std::swap` is what you ship.

Constraints: works for any `int` values including `INT_MIN`/`INT_MAX`. Must be safe when both references alias the same variable.

Example: after `xorSwap(a, b)` with `a=3, b=5` you get `a=5, b=3`. Example: `a=-2, b=4` becomes `a=4, b=-2`.

hint: XOR is its own inverse: `x ^ y ^ y == x`. Three chained XORs move each value into the other slot.
hint: `a ^= b; b ^= a; a ^= b;` leaves `a` and `b` swapped — trace it symbolically.
hint: If both references point at the *same* object the chain XORs it to zero; guard with `if (&a == &b) return;`.

```cpp
// starter
#include <cstdint>
void xorSwap(int& a, int& b);
```

```cpp
void xorSwap(int& a, int& b) {
    if (&a == &b) return;   // aliasing would zero the object
    a ^= b;
    b ^= a;   // b = b ^ (a ^ b) = a
    a ^= b;   // a = (a ^ b) ^ a = b
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    { int a = 3, b = 5; xorSwap(a, b); if (!(a == 5 && b == 3)) { std::puts("case1"); return 1; } }
    { int a = -2, b = 4; xorSwap(a, b); if (!(a == 4 && b == -2)) { std::puts("case2"); return 1; } }
    { int a = 0, b = 0; xorSwap(a, b); if (!(a == 0 && b == 0)) { std::puts("case3"); return 1; } }
    { int a = INT_MIN, b = INT_MAX; xorSwap(a, b); if (!(a == INT_MAX && b == INT_MIN)) { std::puts("case4"); return 1; } }
    { int a = 7, b = 7; xorSwap(a, b); if (!(a == 7 && b == 7)) { std::puts("case5"); return 1; } }
    { int a = 42; xorSwap(a, a); if (a != 42) { std::puts("case6-alias"); return 1; } }  // must survive aliasing
    std::puts("PASS");
}
```

**Editorial:** XOR is associative, commutative, and self-inverse (`v ^ v == 0`, `v ^ 0 == v`). After `a ^= b`, `a` holds `a0 ^ b0`. Then `b ^= a` makes `b = b0 ^ a0 ^ b0 = a0`, and `a ^= b` makes `a = (a0 ^ b0) ^ a0 = b0` — swapped, no temporary. XOR on `int` is defined for all bit patterns including `INT_MIN`, unlike an add/subtract swap which can overflow. The one trap: if `&a == &b`, the first XOR zeroes the shared object and it is lost, so the `&a == &b` guard is mandatory. O(1), branchless apart from that one-time aliasing check.
