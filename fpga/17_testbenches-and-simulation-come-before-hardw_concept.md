## fact: Testbenches and simulation come before hardware
tags: verification, workflow
track: fpga

You never debug on the chip first. A **testbench** is HDL (or C++) that drives stimulus into your design and checks its outputs in a **simulator** — Verilator, ModelSim/Questa, or Vivado's built-in simulator. Simulation gives full visibility into every signal on every cycle, which real silicon cannot, and it avoids the hours-long place-and-route build you'd pay just to observe a bug.

Because hardware bugs are so expensive to iterate on, verification often takes more effort than the design itself. HFT teams simulate against captured real market data to prove a parser or strategy is bit-exact before it ever touches the fabric.
