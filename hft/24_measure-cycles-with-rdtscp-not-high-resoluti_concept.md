## fact: Measure cycles with rdtscp, not high_resolution_clock
tags: measurement, rdtsc, benchmarking
track: hft

For nanosecond timing, read the CPU timestamp counter with **`rdtsc`/`rdtscp`**. Plain `rdtsc` can be reordered by out-of-order execution, so you fence it: `lfence; rdtsc`, or `rdtscp` (which waits for prior instructions to retire) plus `lfence`. `rdtscp` also returns the core id so you can detect a mid-measurement migration.

`std::chrono::high_resolution_clock` is convenient but coarser and, on some implementations, just an alias for `system_clock` (subject to NTP/wall-clock jumps) — prefer `steady_clock` for durations. Its resolution and call overhead exceed a fenced TSC read.

Caveat: on modern **invariant-TSC** CPUs the counter ticks at a *constant reference rate* (unaffected by frequency scaling), so convert ticks→ns with the TSC frequency, not the current core clock. Pin the thread — TSC values are only comparable on the same core.
