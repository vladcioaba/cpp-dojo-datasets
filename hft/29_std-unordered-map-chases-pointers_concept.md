## fact: std::unordered_map chases pointers
tags: hash-map, data-structures, cache
track: hft

The standard requires `std::unordered_map` to behave like **separate chaining with reference stability** (references survive rehash), forcing a **node-per-element** layout: each lookup hashes, then chases a pointer to a scattered heap node — a likely cache miss. Great semantics, poor locality.

**Open-addressing / flat hash maps** (`absl::flat_hash_map`, `boost::unordered_flat_map`, `ankerl::unordered_dense`) store keys/values inline in a contiguous array and probe within it, so a lookup is usually one cache line — often several times faster. Trade-off: pointers/iterators can invalidate on rehash, and erase is a touch more involved. On the hot path, flat maps (or even a sorted `std::vector` with binary search for small/static sets) usually win.
