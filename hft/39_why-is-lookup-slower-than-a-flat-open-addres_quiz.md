## quiz: Why is `std::unordered_map` lookup slower than a flat/open-addressing map?
tags: containers, cache, hash-map
track: hft

- [ ] `unordered_map` is required to use a weaker default hash function
- [x] The standard mandates a node-based design: buckets hold chains of separately heap-allocated nodes, so each probe chases a pointer to a scattered address (cache miss); an open-addressing map stores entries in one contiguous array
- [ ] `unordered_map` is forbidden from using SIMD internally
- [ ] Its maximum load factor is capped at 0.5

> `std::unordered_map` must give reference/pointer stability, which forces separately allocated nodes linked into buckets. A lookup hashes to a bucket and then pointer-chases the chain, and those nodes sit at random heap addresses — one cache miss per hop, plus per-insert allocation. A flat/open-addressing map (e.g. a Swiss-table style `flat_hash_map`) keeps keys in a contiguous array and probes neighbors that are already in cache, which is why it usually wins on the hot path.
