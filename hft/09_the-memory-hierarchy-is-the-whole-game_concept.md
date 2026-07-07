## fact: The memory hierarchy is the whole game
tags: cache, memory, latency
track: hft

Modern x86-64 moves memory in **64-byte cache lines**, never single bytes. Access latencies span orders of magnitude: **L1 ~1 ns (~4 cycles)**, **L2 ~3-4 ns (~12 cycles)**, **L3 ~10-20 ns (~40 cycles)**, **DRAM ~60-100 ns (hundreds of cycles)**. One L1→DRAM miss can cost ~100 ns — time enough to retire hundreds of instructions.

Design for **spatial locality** (touch bytes that share a line) and **temporal locality** (reuse hot data before it's evicted). Struct-of-arrays often beats array-of-structs because you only pull the fields you iterate over into cache.

The hardware prefetcher detects sequential/strided access and fetches lines ahead of use, which is why linear scans over `std::vector` fly. For irregular access you can hint with `__builtin_prefetch(ptr)` (or `_mm_prefetch`), but measure — a bad prefetch wastes bandwidth and evicts useful lines.
