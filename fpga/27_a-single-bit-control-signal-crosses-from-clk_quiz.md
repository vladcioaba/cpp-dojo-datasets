## quiz: A single-bit control signal crosses from clk_a into clk_b. What is the standard safe technique?
tags: cdc, timing
track: fpga

- [ ] Register it once in clk_b
- [x] Pass it through two chained flip-flops in the clk_b domain (a 2-FF synchronizer)
- [ ] Combinationally AND it with clk_b
- [ ] Nothing special is needed as long as both clocks are the same frequency

> A single register can latch a metastable value and pass it downstream. Two chained flops give the first time to settle before the second samples, pushing the failure probability astronomically low. Equal frequency doesn't help unless the clocks are also phase-aligned. Multi-bit buses need Gray coding, a handshake, or an async FIFO instead.
