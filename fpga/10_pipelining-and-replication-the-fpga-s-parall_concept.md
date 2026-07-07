## fact: Pipelining and replication — the FPGA's parallelism
tags: pipelining, parallelism, timing
track: fpga

**Pipelining** breaks a long combinational path into stages separated by registers. Each stage is shorter, so the clock can run faster (higher **Fmax**), and once full the pipeline emits a result **every cycle**. The price is **latency** — a result now takes N cycles to cross N stages — but throughput soars.

The other axis is **spatial parallelism**: **unrolling** a loop into replicated hardware and **replicating** whole datapaths so N things happen physically at once instead of N times in sequence. A CPU at 4 GHz still runs only a few instructions per cycle over one shared ALU, with jitter from branches and cache misses; a 300 MHz FPGA can lay **hundreds of operations side by side**, all firing each cycle with identical latency. For a wide, fixed computation the lower clock simply doesn't matter.
