## quiz: Roughly, tick-to-trade latency for a tuned FPGA path vs a kernel-bypass software path is:
tags: hft, latency
track: fpga

- [ ] Both are in the millisecond range
- [ ] FPGA in microseconds, software in milliseconds
- [x] FPGA in tens-to-hundreds of nanoseconds, software in single-digit microseconds
- [ ] FPGA in picoseconds, software in nanoseconds

> A hardware path processes the message as a circuit in tens to low hundreds of nanoseconds; even kernel-bypass software sits in the single-digit-microsecond range — roughly an order of magnitude apart. Picosecond tick-to-trade isn't real, and milliseconds are far too slow to compete.
