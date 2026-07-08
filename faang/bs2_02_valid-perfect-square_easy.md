## challenge: Valid Perfect Square
tags: binary-search, math
track: faang
difficulty: easy

Given a positive integer `num`, return `true` if it is a perfect square (the square of some integer) and `false` otherwise. You must not use any built-in square-root function such as `sqrt`.

Constraints: `1 <= num <= 2^31 - 1`.

Example: `num = 16` → `true` (`4 * 4`). Example: `num = 14` → `false`. Example: `num = 1` → `true`.

hint: The candidate roots `1, 2, ..., num` are sorted, and `mid * mid` is monotonically increasing — a perfect setup for binary search.
hint: Search for a `mid` whose square equals `num`; steer `lo`/`hi` by comparing `mid * mid` against `num`.
hint: `mid * mid` overflows 32 bits when `mid` is large — do the multiplication in `long long`.

```cpp
// starter
bool isPerfectSquare(int num);
```

```cpp
bool isPerfectSquare(int num) {
    long long lo = 1, hi = num;
    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        long long sq = mid * mid;
        if (sq == num) return true;
        if (sq < num) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (isPerfectSquare(16)         != true)  { std::puts("case1"); return 1; }
    if (isPerfectSquare(14)         != false) { std::puts("case2"); return 1; }
    if (isPerfectSquare(1)          != true)  { std::puts("case3"); return 1; }
    if (isPerfectSquare(808201)     != true)  { std::puts("case4"); return 1; }   // 899^2
    if (isPerfectSquare(2147395600) != true)  { std::puts("case5"); return 1; }   // 46340^2
    if (isPerfectSquare(2147483647) != false) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The map `mid -> mid * mid` is strictly increasing over the candidate roots, so binary search `[1, num]` for a root whose square hits `num`. Compare `mid * mid` with `num` to collapse the window. The only trap is overflow: for `num` near `2^31`, `mid` reaches ~46341 and `mid * mid` exceeds a 32-bit `int`, so compute the square in `long long`. O(log num) time, O(1) space.
