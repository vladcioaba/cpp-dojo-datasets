## fact: Inlining is the mother of optimizations
tags: inlining, lto, pgo
track: hft

Inlining removes call overhead but, more importantly, **exposes the callee to the caller's optimizer** — constant propagation, dead-code elimination, and vectorization across the old boundary. That's why hot functions should stay inlinable (small, in headers, non-virtual). The `inline` keyword is mostly about ODR/linkage, not a command; force it with `[[gnu::always_inline]]` when the heuristic is wrong (and `[[gnu::noinline]]` to keep cold paths out of the I-cache).

**LTO** (link-time optimization) inlines and optimizes **across translation units** at link time, recovering what separate compilation lost. **PGO** (profile-guided optimization) feeds a real workload's profile back to the compiler so it co-locates hot code, predicts branches, and inlines what actually runs hot. Both are standard for the last few percent of a latency-critical binary.
