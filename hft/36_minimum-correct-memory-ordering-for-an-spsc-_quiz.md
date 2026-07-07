## quiz: Minimum correct memory ordering for an SPSC flag handoff
tags: concurrency, memory-order, atomics
track: hft

```cpp
int data;
std::atomic<bool> ready{false};
// producer:            // consumer:
data = 42;              while (!ready.load(/* ? */)) {}
ready.store(true, /* ? */);   use(data);
```

- [ ] `relaxed` on both — the atomic is enough
- [x] `release` on the store, `acquire` on the load
- [ ] `acquire` on the store, `release` on the load
- [ ] `seq_cst` on both is the only correct choice

> The store-release publishes every write sequenced before it (including `data = 42`); the load-acquire, once it observes `true`, is guaranteed to see those writes — that release/acquire pair creates the happens-before edge. `relaxed` gives no ordering, so the consumer could read a stale/torn `data`. The acquire/release pair is the *minimum* correct ordering; `seq_cst` also works but is stronger (and slower) than needed here.
