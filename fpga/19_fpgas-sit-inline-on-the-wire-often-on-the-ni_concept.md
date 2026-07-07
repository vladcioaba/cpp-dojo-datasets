## fact: FPGAs sit inline on the wire, often on the NIC
tags: hft, architecture
track: fpga

The lowest-latency designs put the FPGA **inline with the network** — frequently on a smart **NIC** so packets hit the fabric the instant they arrive, before any host CPU is involved. The FPGA parses the market-data feed, evaluates logic, and can emit an order **wire-to-wire** without a round trip to software.

Typical hardware functions in the fast path are **feed handlers** that decode and normalize market data, **pre-trade risk checks**, and **order entry**. Slower, more complex logic — strategy calibration, position management — stays in software on the host, which parameterizes or reconfigures the FPGA. This split of a hardware fast path and a software slow path is the standard HFT architecture.
