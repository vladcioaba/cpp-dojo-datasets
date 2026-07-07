## quiz: Beyond raw average speed, why do HFT firms value FPGAs for the fast path?
tags: hft, determinism
track: fpga

- [ ] They consume less electricity than any CPU
- [x] Deterministic latency — no OS scheduler, cache misses, or GC, so every message takes the same number of cycles, keeping the tail tight
- [ ] They can execute arbitrary Python at line rate
- [ ] They never need to be reprogrammed once deployed

> The latency tail is what gets you picked off. An FPGA circuit takes an identical cycle count per message — no jitter from interrupts, caches, or garbage collection. A predictable 200 ns can beat an average 1 µs that spikes to 50 µs. Power isn't the driver, and FPGAs don't run Python.
