## fact: The FPGA-vs-software tradeoff is real
tags: hft, tradeoffs
track: fpga

FPGAs are not free speed. Development is slow and specialized: HDL expertise is scarce, builds take hours, debugging is harder than software, and changing a strategy can mean re-synthesizing and re-timing the whole design. Software iterates in seconds; FPGA turnarounds are measured in hours to days.

So the decision is economic, not just technical. Put in **hardware** what pays for itself in latency and determinism — the innermost tick-to-trade loop, feed handling, risk gates — and keep in **software** whatever needs flexibility: research, calibration, and less latency-critical strategies. The winning systems are hybrids that place each piece where it belongs.
