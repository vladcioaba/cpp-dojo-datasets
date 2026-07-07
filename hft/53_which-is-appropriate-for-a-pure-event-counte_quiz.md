## quiz: Which `memory_order` is appropriate for a pure event counter?
tags: concurrency, memory-order, relaxed
track: hft

```cpp
std::atomic<uint64_t> events{0};
// many threads:
events.fetch_add(1, /* ? */);
// read once, at shutdown
```

- [ ] It must be `seq_cst` or the total will be wrong
- [x] `relaxed` is sufficient: the RMW is still atomic (no lost updates); you just don't need it ordered against other memory operations
- [ ] `relaxed` can lose increments under contention
- [ ] `release` is required on every `fetch_add`

> `memory_order_relaxed` still guarantees the increment is a single atomic read-modify-write, so no updates are lost no matter how many threads contend. What it drops is *ordering* relative to other variables — and a standalone counter that nobody uses to publish other data needs no such ordering. It is the cheapest correct choice; `seq_cst` would add global-ordering fences you do not need here.
