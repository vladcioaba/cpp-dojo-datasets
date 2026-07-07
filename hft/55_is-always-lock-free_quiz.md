## quiz: Is `std::atomic<T>` always lock-free?
tags: concurrency, atomics, lock-free
track: hft

```cpp
struct Big { double a, b, c; };   // 24 bytes
std::atomic<Big> x;
```

- [ ] Every `std::atomic` specialization is lock-free by definition
- [x] For a type too large for a single hardware atomic instruction (a 24-byte struct), the implementation falls back to a hidden lock; check `is_lock_free()` — a "lock-free" atomic that isn't defeats the purpose on the hot path
- [ ] `std::atomic<Big>` fails to compile
- [ ] It is lock-free but every load allocates

> `std::atomic` works for any trivially copyable type, but the CPU can only perform a truly atomic operation on operands up to its widest atomic instruction (8 bytes, or 16 with `cmpxchg16b`). A 24-byte struct exceeds that, so libstdc++/libc++ guard it with an internal lock (a mutex or a striped lock table), and reads/writes are no longer wait-free. Query `x.is_lock_free()` or the compile-time `std::atomic<Big>::is_always_lock_free`; a surprise lock in your supposedly lock-free fast path is a latency trap.
