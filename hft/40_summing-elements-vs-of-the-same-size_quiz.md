## quiz: Summing elements: `std::vector<int>` vs `std::list<int>` of the same size
tags: containers, cache, locality
track: hft

- [x] `vector` is much faster: contiguous storage lets the hardware prefetcher stream the data with few cache misses; `list` nodes are scattered, costing a cache miss per hop
- [ ] `list` is faster because traversal is O(1) per node
- [ ] They are equally fast since both are O(N)
- [ ] `list` is faster because it skips bounds checking

> Both are O(N) in the abstract, but wall-clock time is dominated by the memory system. A `vector`'s elements are laid out contiguously, so sequential access triggers the prefetcher and touches each cache line once. A `list` allocates each node independently, so traversal is a chain of pointer dereferences to unpredictable addresses — typically a cache miss per element. Contiguity, not asymptotics, decides this.
