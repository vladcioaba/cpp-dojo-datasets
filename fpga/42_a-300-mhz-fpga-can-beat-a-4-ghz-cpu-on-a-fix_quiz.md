## quiz: A 300 MHz FPGA can beat a 4 GHz CPU on a fixed feed-parsing pipeline mainly because:
tags: parallelism, tradeoffs
track: fpga

- [ ] Its clock is actually faster than the CPU's
- [x] It lays hundreds of operations physically side by side, all firing every cycle with constant latency — spatial parallelism beats the CPU's shared, sequential datapath
- [ ] It has a much larger cache than the CPU
- [ ] It uses floating-point where the CPU is stuck with integers

> Despite the lower clock, the FPGA does in one cycle what a CPU needs many sequential instructions for, and every message takes the same time. The advantage is spatial parallelism and determinism, not clock speed, cache size, or numeric type.
