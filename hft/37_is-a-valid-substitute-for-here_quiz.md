## quiz: Is `volatile` a valid substitute for `std::atomic` here?
tags: concurrency, volatile, atomics
track: hft

```cpp
volatile int ready = 0;
volatile int data  = 0;
// thread A: data = 42; ready = 1;
// thread B: while (!ready) {}  use(data);
```

- [ ] `volatile` makes the accesses atomic and correctly ordered across threads
- [x] `volatile` provides neither atomicity nor inter-thread ordering — this is a data race (UB); use `std::atomic`
- [ ] It is correct but simply slower than `std::atomic`
- [ ] It works on x86 but the `int` may still be torn

> `volatile` only tells the compiler not to elide or fold the accesses (it was designed for memory-mapped I/O). It creates no happens-before relationship and issues no hardware fences, so the CPU/compiler can still reorder `data` relative to `ready` and thread B may never observe the update. Concurrent conflicting access without atomics is a data race and therefore undefined behavior. Use `std::atomic` with the right memory order.
