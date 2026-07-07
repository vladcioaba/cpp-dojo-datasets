## quiz: What distinguishes "full hardware" tick-to-trade from kernel bypass?
tags: hft, architecture
track: fpga

- [ ] Kernel bypass runs the entire strategy inside the OS kernel
- [x] In full hardware the FPGA parses, decides, and emits the order with no CPU in the path; kernel bypass still runs the logic as software in userspace
- [ ] They are two names for the same technique
- [ ] Full hardware is slower but much easier to modify

> Kernel bypass (Onload, DPDK) skips the OS network stack, but the strategy still runs on a CPU. Full hardware keeps the entire hot path inside the FPGA, so a matching packet can trigger an order without a CPU touching it — faster and more deterministic, but harder to change.
