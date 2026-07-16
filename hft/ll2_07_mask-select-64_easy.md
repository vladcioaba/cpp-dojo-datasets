## challenge: Branchless select with an all-ones mask
tags: branch-prediction, bit-tricks, hot-path
track: hft
difficulty: easy

Implement `uint64_t select64(bool take_a, uint64_t a, uint64_t b)` that returns `a` when `take_a` is true and `b` otherwise — with no `if`, no ternary, no branches at all. Build a mask that is all-ones when the condition is true and all-zeros when false, then combine. This is the hand-rolled `cmov`: when the condition is unpredictable (e.g. "did the order cross the spread?"), a mispredicted branch costs more than the whole computation.

Constraints: any `uint64_t` values for `a` and `b`; straight-line code — only arithmetic and bitwise ops on the bool and the operands.

Example: `select64(true, 10, 20) == 10`, `select64(false, 10, 20) == 20`, `select64(true, 0xFFFFFFFFFFFFFFFF, 0) == 0xFFFFFFFFFFFFFFFF`.

hint: A `bool` converts to integer 0 or 1; you need it as 0 or `0xFFFF...F` — unsigned negation does it: `0 - 1` wraps to all ones.
hint: `uint64_t m = 0ull - static_cast<uint64_t>(take_a);` then pick with `(a & m) | (b & ~m)`.
hint: An equivalent with one fewer op: `b ^ ((a ^ b) & m)` — when `m` is all ones the XORs cancel to `a`, when zero it stays `b`.

```cpp
// starter
#include <cstdint>
uint64_t select64(bool take_a, uint64_t a, uint64_t b);
```

```cpp
uint64_t select64(bool take_a, uint64_t a, uint64_t b) {
    uint64_t m = 0ull - static_cast<uint64_t>(take_a);  // true -> all ones, false -> 0
    return (a & m) | (b & ~m);
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
//__USER__
int main() {
    struct { bool c; uint64_t a, b, want; } cases[] = {
        {true,  10u, 20u, 10u},
        {false, 10u, 20u, 20u},
        {true,  0xFFFFFFFFFFFFFFFFull, 0u, 0xFFFFFFFFFFFFFFFFull},
        {false, 0xFFFFFFFFFFFFFFFFull, 0u, 0u},
        {true,  0u, 0xFFFFFFFFFFFFFFFFull, 0u},
        {false, 0u, 0xFFFFFFFFFFFFFFFFull, 0xFFFFFFFFFFFFFFFFull},
        {true,  0xDEADBEEFCAFEBABEull, 0x0123456789ABCDEFull, 0xDEADBEEFCAFEBABEull},
        {false, 0xDEADBEEFCAFEBABEull, 0x0123456789ABCDEFull, 0x0123456789ABCDEFull},
        {true,  7u, 7u, 7u},   // equal operands
    };
    for (auto& c : cases) {
        uint64_t got = select64(c.c, c.a, c.b);
        if (got != c.want) {
            std::printf("select64(%d,%llx,%llx)=%llx want %llx\n",
                        (int)c.c, (unsigned long long)c.a, (unsigned long long)c.b,
                        (unsigned long long)got, (unsigned long long)c.want);
            return 1;
        }
    }
    std::puts("PASS");
}
```

**Editorial:** The core trick is manufacturing a full-width mask from a 1-bit condition: `bool` converts to 0 or 1, and unsigned negation `0 - x` wraps modulo 2^64, so 1 becomes `0xFFFF...F` and 0 stays 0 (perfectly defined for unsigned types — no UB). With the mask in hand, `(a & m) | (b & ~m)` muxes bitwise between the operands; the XOR form `b ^ ((a ^ b) & m)` does it in three ops. Why bother when `cond ? a : b` exists? For a plain value ternary the compiler usually emits `cmov` anyway — but "usually" is doing heavy lifting: surround it with other code and the optimizer may prefer a branch, and on an unpredictable condition each mispredict flushes ~15–20 cycles of pipeline. The mask form *guarantees* straight-line code: constant latency, no predictor state, no timing variation — which also makes it a staple of constant-time cryptography, where a data-dependent branch is an information leak. Know both spellings; reach for the mask when the condition is coin-flip unpredictable and the latency budget is single-digit nanoseconds.
