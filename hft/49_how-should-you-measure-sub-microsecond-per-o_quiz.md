## quiz: How should you measure sub-microsecond per-operation latency?
tags: measurement, latency, rdtsc
track: hft

- [ ] Take the mean of `std::chrono::system_clock`; it is monotonic and nanosecond-accurate
- [x] Use a high-resolution monotonic source (`rdtsc` / `steady_clock`) and report tail percentiles (p99, p99.9, max), not the mean — latency distributions are heavy-tailed
- [ ] The mean latency fully characterizes the tail
- [ ] `system_clock` is best because it can step backwards to correct for drift

> `system_clock` is wall-clock time and can jump backward or forward (NTP, adjustments), so it is wrong for measuring elapsed durations — use `steady_clock` or, for the lowest overhead, `rdtsc` (with care: serialize it, rely on invariant TSC, and beware core migration and cycle-to-nanosecond conversion). And a single number is not enough: HFT cares about the tail, so report p99/p99.9/max, not the mean, which hides exactly the spikes that hurt.
