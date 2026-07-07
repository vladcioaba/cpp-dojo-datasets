## fact: FPGA vs ASIC vs CPU — the reconfigurable middle ground
tags: fundamentals, tradeoffs
track: fpga

A **CPU** is fully general but executes instructions sequentially over a shared datapath — flexible, but high-latency and subject to caches, branch prediction, and OS jitter. An **ASIC** is a fully custom chip: fastest and most power-efficient, but a multi-million-dollar, multi-month tape-out that can never be changed once made.

An **FPGA** sits between them. You get hardware parallelism and near-ASIC latency, yet the fabric is **reconfigurable** in seconds by loading a new bitstream. You pay for that flexibility in clock speed (FPGAs run at hundreds of MHz, not GHz), silicon area, and power. For HFT the payoff is doing the work as a fixed circuit instead of a stream of instructions.
