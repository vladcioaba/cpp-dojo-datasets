## quiz: Why put pre-trade risk checks (size, price, position limits) inside the FPGA?
tags: hft, risk
track: fpga

- [ ] Because running risk checks in software is illegal
- [x] They become a few comparators the order passes in a couple of cycles — unconditional and deterministic, with almost no latency cost
- [ ] Because an FPGA physically cannot send an order without them
- [ ] To let each order opt out of the checks individually

> Hardware risk gates are cheap (just comparators) and guarantee no order reaches the exchange without passing — safety and speed together, unlike software checks that add microseconds. They are mandatory, not opt-out, and software risk systems are perfectly legal and common too.
