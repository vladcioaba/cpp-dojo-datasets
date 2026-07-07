## quiz: Many threads run `counter++` on a shared `long long`
tags: concurrency, data-race, atomics
track: hft

```cpp
long long counter = 0;   // shared, non-atomic
// many threads: counter++;
```

- [ ] Fine, because aligned 64-bit stores are atomic on x86-64
- [ ] Fine, as long as exactly one thread ever writes
- [x] `counter++` is a read-modify-write data race — concurrent conflicting access without atomics is UB, and updates are lost
- [ ] Marking it `volatile` would make `++` atomic

> `counter++` is three steps: load, add, store. Two threads can both load the same value and each store back the same `+1`, losing an update. Even setting aside lost updates, the C++ memory model says two threads accessing the same non-atomic object where at least one writes is a data race — undefined behavior — so the compiler is free to assume it never happens. `volatile` does not make `++` atomic. Use `std::atomic<long long>` with `fetch_add`.
