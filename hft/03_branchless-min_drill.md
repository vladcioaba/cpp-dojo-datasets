## challenge: Branchless min
tags: branchless, bit-tricks
track: hft

Branch mispredicts cost ~15-20 cycles. Compute the minimum of two ints with a bit trick instead of a branch: `b ^ ((a ^ b) & -(a < b))`. Implement `int bmin(int a, int b)` returning the smaller. (The compiler often does this for you — but interviewers ask you to derive it.)

hint: The trick removes the conditional jump so the CPU cannot mispredict it — the whole result comes from turning a boolean into a bitmask.
hint: `-(a < b)` is either 0 (all bits clear) or -1 (all bits set), which selects between `a` and `b` using XOR.

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

**Editorial:** `a < b` evaluates to 0 or 1, and negating it produces an all-zero or all-ones mask. `b ^ ((a ^ b) & mask)` collapses to `b` when the mask is 0 and to `a` when it is all-ones, choosing the minimum with no branch to mispredict. Constant O(1) work.
