## fact: Type-pun with bit_cast/memcpy, never reinterpret_cast
tags: type-punning, bit-cast, aliasing, restrict
track: hft

Reading a `float`'s bytes via `*reinterpret_cast<int*>(&f)` violates **strict aliasing** and is undefined — the optimizer may assume the two never alias and reorder or elide your load. The correct, zero-cost tools:

```cpp
float f = 1.0f;
int i = std::bit_cast<int>(f);        // C++20, constexpr, sizes must match
int j; std::memcpy(&j, &f, sizeof j); // pre-C++20; folded to a register move
```

`std::bit_cast`/`memcpy` say "reinterpret these bytes" without lying to the aliasing analysis, and compilers lower them to a plain move — no actual copy. Conversely, when you *promise* pointers don't overlap, `restrict` (`__restrict` in C++) lets the compiler drop reload/aliasing guards and vectorize — but a false `restrict` promise is itself UB.
