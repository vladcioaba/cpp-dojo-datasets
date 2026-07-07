## challenge: Climbing Stairs
tags: dynamic-programming, math
track: faang
difficulty: easy

You are climbing a staircase with `n` steps. Each time you can climb `1` or `2` steps. In how many distinct ways can you reach the top?

Constraints: `1 <= n <= 45`.

Example: `n = 2` → `2` (`1+1`, `2`). Example: `n = 3` → `3` (`1+1+1`, `1+2`, `2+1`). Example: `n = 5` → `8`.

hint: The number of ways to reach step n depends only on the ways to reach steps n-1 and n-2.
hint: That recurrence is exactly Fibonacci — you only need to keep the last two values.

```cpp
// starter
int climbStairs(int n);
```

```cpp
int climbStairs(int n) {
    int a = 1, b = 1;
    for (int i = 0; i < n; ++i) {
        int c = a + b;
        a = b;
        b = c;
    }
    return a;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    int want[] = {1, 1, 2, 3, 5, 8, 13, 21}; // index == n, for n = 0..7
    for (int n = 1; n <= 7; ++n) {
        int got = climbStairs(n);
        if (got != want[n]) { std::printf("climbStairs(%d)=%d want %d\n", n, got, want[n]); return 1; }
    }
    if (climbStairs(45) != 1836311903) { std::puts("case45"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The last move onto step n is either a 1-step from n-1 or a 2-step from n-2, so ways(n) = ways(n-1) + ways(n-2). Roll two variables forward instead of storing the whole table. O(n) time, O(1) space.
