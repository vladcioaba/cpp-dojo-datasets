## quiz: Your design fails timing — the critical path is too long for the target clock. Which change most directly helps?
tags: timing, pipelining
track: fpga

- [ ] Switch every assignment from non-blocking to blocking
- [x] Insert pipeline registers to split the long combinational path into shorter stages
- [ ] Change the reset from synchronous to asynchronous
- [ ] Move the logic into BRAM

> Registering intermediate results shortens the longest combinational path, raising Fmax — at the cost of extra latency cycles. Assignment style and reset type don't change path delay, and BRAM is memory, not a place to "put logic."
