## fact: std::vector beats std::list almost always
tags: containers, vector, cache
track: hft

`std::list` is a doubly-linked list: every node is a **separate allocation**, and traversal **pointer-chases** across scattered memory — a cache miss per element. `std::vector` is contiguous, so iteration streams through cache and the prefetcher loves it. Even mid-insertion (a memmove) usually beats `list`'s "cheap" splice once cache misses dominate — for realistic sizes `vector` wins on nearly every operation. Prefer it by default and `reserve()` to avoid reallocation.

**Small-buffer optimization (SBO/SSO)**: `std::string` and small-vector types (`boost::small_vector`, `llvm::SmallVector`) store short contents **inline** in the object, dodging the heap for the common small case — which is why a short `std::string` never calls `malloc`. Choose containers by memory layout and allocation behavior, not just asymptotic complexity.
