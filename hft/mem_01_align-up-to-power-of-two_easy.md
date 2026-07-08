## challenge: Align up to a power-of-two boundary
tags: bit-tricks, alignment, power-of-two
track: hft
difficulty: easy

Every custom allocator needs one primitive: round a size (or address) up to the next multiple of an alignment. Implement `align_up(n, a)` returning the smallest value `>= n` that is a multiple of `a`, where `a` is a power of two. Do it in O(1) with bit tricks — no loops, no division.

Constraints: `a` is a power of two (`1, 2, 4, 8, ...`); the aligned result fits in `std::size_t`.

Example: `align_up(1, 8) == 8`, `align_up(8, 8) == 8`, `align_up(13, 16) == 16`, `align_up(17, 16) == 32`, `align_up(0, 8) == 0`.

hint: For a power-of-two `a`, `a - 1` is a mask of the low bits; a value is aligned exactly when those low bits are all zero.
hint: Add `a - 1` first so anything not already aligned spills into the next boundary, then clear the low bits with `& ~(a - 1)`.
hint: Avoid `%` and division — on the hot path the mask form `(n + a - 1) & ~(a - 1)` is a couple of instructions.

```cpp
// starter
#include <cstddef>
std::size_t align_up(std::size_t n, std::size_t a);
```

```cpp
std::size_t align_up(std::size_t n, std::size_t a) {
    return (n + (a - 1)) & ~(a - 1);
}
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
//__USER__
int main() {
    struct { std::size_t n, a, want; } cases[] = {
        {0, 8, 0}, {1, 8, 8}, {7, 8, 8}, {8, 8, 8}, {9, 8, 16},
        {13, 16, 16}, {16, 16, 16}, {17, 16, 32},
        {1, 1, 1}, {1000, 64, 1024}, {4096, 4096, 4096},
    };
    for (auto& c : cases) {
        std::size_t got = align_up(c.n, c.a);
        if (got != c.want) { std::printf("align_up(%zu,%zu)=%zu want %zu\n", c.n, c.a, got, c.want); return 1; }
        if (got % c.a != 0) { std::puts("result not aligned"); return 1; }
        if (got < c.n)      { std::puts("result smaller than input"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** For a power-of-two `a`, the low `log2(a)` bits of any aligned value are zero, and `a - 1` is exactly the mask of those bits. Adding `a - 1` pushes any unaligned `n` up into the next boundary's range without overshooting past it; the subsequent `& ~(a - 1)` clears the low bits to land precisely on the boundary. Already-aligned inputs are unchanged because adding `a - 1` stays below the next multiple. This branchless, division-free form is the workhorse inside arena and stack allocators.
