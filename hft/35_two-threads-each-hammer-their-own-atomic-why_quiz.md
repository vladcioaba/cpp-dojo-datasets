## quiz: Two threads each hammer their own atomic — why is this slow?
tags: concurrency, cache, false-sharing
track: hft

```cpp
struct Counters {
    std::atomic<long> a;   // thread 1 does c.a.fetch_add(1) in a tight loop
    std::atomic<long> b;   // thread 2 does c.b.fetch_add(1) in a tight loop
};
Counters c;
```

- [ ] Atomic increments are inherently serialized across all cores
- [x] `a` and `b` live in the same 64-byte cache line, so the two cores keep invalidating each other's copy (false sharing); give each its own line with `alignas(64)`
- [ ] The compiler reorders the two increments into a single contended one
- [ ] `std::atomic<long>` takes a global lock internally

> Even though the threads touch different variables, `a` and `b` share one cache line. Every write forces the line into the writing core's cache in Modified state, invalidating the other core's copy — the line ping-pongs on the interconnect. Padding/aligning each counter to its own 64-byte line (`alignas(64) std::atomic<long> a;`) eliminates the contention.
