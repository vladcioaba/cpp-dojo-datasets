## fact: An FPGA is a chip you rewire, not one you program
tags: fundamentals, hardware
track: fpga

An **FPGA** (Field-Programmable Gate Array) is a sea of digital logic you configure *after* manufacturing. Rather than executing instructions like a CPU, you describe a circuit and load a **bitstream** that makes the chip physically become that circuit. The primitives are **LUTs** (look-up tables) that implement any Boolean function of their inputs, **flip-flops** that each store one bit, and a programmable **routing fabric** of wires connecting them. Vendors group these into **CLBs** (Configurable Logic Blocks; Intel/Altera calls them LABs); a modern LUT typically has 6 inputs backed by a 64-entry truth-table SRAM.

Two kinds of hardened blocks round out the fabric. **BRAM** (Block RAM) is dedicated dual-port memory — kilobits per block, thousands of blocks per device — for buffers, FIFOs, and tables. **DSP slices** are hardened multiply-accumulate units (Xilinx calls them DSP48) that do fixed-point arithmetic far more efficiently than logic built from LUTs.

Every design is a **budget** across four resources — LUTs, flip-flops, BRAM, and DSPs. Exhaust any one and the design won't fit, even if the others sit idle.
