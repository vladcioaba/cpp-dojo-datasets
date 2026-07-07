## challenge: Branchless min
tags: branchless, bit-tricks
track: hft

Branch mispredicts cost ~15-20 cycles. Compute the minimum of two ints with a bit trick instead of a branch: `b ^ ((a ^ b) & -(a < b))`. Implement `int bmin(int a, int b)` returning the smaller. (The compiler often does this for you — but interviewers ask you to derive it.)

```cpp
int bmin(int a, int b) {
    return b ^ ((a ^ b) & -(a < b));
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    struct { int a, b, want; } cases[] = {
        {3, 5, 3}, {5, 3, 3}, {-2, 4, -2}, {7, 7, 7},
        {0, -1, -1}, {INT_MAX, 0, 0}, {-100, -100, -100}, {1, INT_MAX, 1},
    };
    for (auto& c : cases) {
        int got = bmin(c.a, c.b);
        if (got != c.want) { std::printf("bmin(%d,%d)=%d want %d\n", c.a, c.b, got, c.want); return 1; }
    }
    std::puts("PASS");
}
```
