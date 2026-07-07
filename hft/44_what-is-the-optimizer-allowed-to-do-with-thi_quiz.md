## quiz: What is the optimizer allowed to do with this function?
tags: undefined-behavior, integer-overflow, optimizer
track: hft

```cpp
int f(int x) {
    return x + 1 > x;   // x is an arbitrary int
}
```

- [ ] Return `false` when `x == INT_MAX`
- [x] Assume signed overflow never happens and compile the body to `return 1;`
- [ ] Insert a runtime overflow check
- [ ] Wrap at `INT_MAX`, so it is implementation-defined

> Signed integer overflow is undefined behavior, so the compiler may assume `x + 1` never overflows — which makes `x + 1 > x` unconditionally true. GCC/Clang fold this to a constant `1`, even for `x == INT_MAX`. (Unsigned arithmetic is defined to wrap, so the analogous unsigned version really can be false.) This is a textbook case of UB enabling a surprising optimization.
