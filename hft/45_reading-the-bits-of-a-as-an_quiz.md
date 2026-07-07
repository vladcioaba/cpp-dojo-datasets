## quiz: Reading the bits of a `float` as an `int`
tags: undefined-behavior, strict-aliasing, type-punning
track: hft

```cpp
float f = 1.0f;
int i = *reinterpret_cast<int*>(&f);   // (A)
int j; std::memcpy(&j, &f, sizeof j);  // (B)
```

- [ ] Both are fine; `reinterpret_cast` is the idiomatic way
- [x] (A) violates strict aliasing and is UB; (B) via `memcpy` (or `std::bit_cast`) is the well-defined way to reinterpret bits
- [ ] (B) is UB because `memcpy` ignores the types
- [ ] Both are UB; a `union` is the only correct option

> Accessing the storage of a `float` through an `int` lvalue breaks the strict-aliasing rule — the compiler assumes an `int*` and a `float*` never refer to the same object and may miscompile accordingly. `std::memcpy` copies the underlying bytes and is fully defined; `std::bit_cast<int>(f)` (C++20) is the modern one-liner and is `constexpr`. Union type-punning is defined in C but not portably in C++.
