## challenge: Overflow-safe midpoint of two integers
tags: bit-tricks, fast-math, overflow
track: hft
difficulty: easy

Computing `(a + b) / 2` is the natural midpoint, but `a + b` can overflow when both are large — the classic binary-search bug. Implement `int64_t avg(int64_t a, int64_t b)` returning `floor((a + b) / 2)` with no intermediate overflow, over the full `int64` range. Use the bitwise identity, not a wider type.

Constraints: `INT64_MIN <= a, b <= INT64_MAX`. Result is `floor((a + b) / 2)`.

Example: `avg(3, 5) == 4`, `avg(2, 7) == 4`, `avg(INT64_MAX, INT64_MAX) == INT64_MAX`, `avg(-4, -1) == -3`, `avg(1, -2) == -1`.

hint: Split the sum into a carry-free part and a carry part: `a + b == (a ^ b) + 2*(a & b)`.
hint: Halving `2*(a & b)` is exact (it is `a & b`), and halving `(a ^ b)` with an arithmetic right shift supplies the floored remainder.
hint: `(a & b) + ((a ^ b) >> 1)` never forms the full sum, so it cannot overflow; in C++20 signed `>>` is guaranteed arithmetic (floor).

```cpp
// starter
#include <cstdint>
int64_t avg(int64_t a, int64_t b);
```

```cpp
int64_t avg(int64_t a, int64_t b) {
    return (a & b) + ((a ^ b) >> 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { int64_t a, b, want; } cases[] = {
        {3,5,4},{2,7,4},{0,0,0},{-4,-1,-3},{1,-2,-1},{-1,-1,-1},{7,7,7},
        {INT64_MAX,INT64_MAX,INT64_MAX},{INT64_MIN,INT64_MIN,INT64_MIN},
        {INT64_MAX,INT64_MIN,-1},{INT64_MAX,0,INT64_MAX/2},
    };
    for (auto& c : cases) {
        int64_t got = avg(c.a, c.b);
        if (got != c.want) { std::printf("avg(%lld,%lld)=%lld want %lld\n",
            (long long)c.a,(long long)c.b,(long long)got,(long long)c.want); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** This is the Bloch binary-search overflow bug fixed with bit math. The identity `a + b == (a ^ b) + ((a & b) << 1)` separates the non-carrying bits (XOR) from the carrying bits (AND, weight 2). Halving the AND term is exact; halving the XOR term with an arithmetic shift supplies the floor. Because the wide sum is never materialized, there is no overflow — pure bit ops, no division.
