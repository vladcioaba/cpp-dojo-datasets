## fact: False sharing — two threads, one cache line
tags: cache, concurrency, false-sharing
track: hft

Two threads writing **different variables that happen to share one 64-byte line** still contend: each write invalidates the line in the other core's cache (MESI), forcing a coherence round-trip. Throughput collapses even though the variables are logically independent.

Fix by aligning hot per-thread data onto its own line with `alignas(std::hardware_destructive_interference_size)` (typically 64 on x86-64; some implementations use 128 to account for adjacent-line prefetch).

```cpp
struct alignas(std::hardware_destructive_interference_size) Counter {
    std::atomic<std::uint64_t> value{0};
};
Counter a, b; // a and b never land on the same cache line
```

The dual constant `std::hardware_constructive_interference_size` tells you the max size to *pack together* on purpose.
