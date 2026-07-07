## quiz: What is metastability?
tags: timing, fundamentals
track: fpga

- [ ] A design that consumes too many DSP slices
- [ ] A permanent stuck-at fault inside a flip-flop
- [x] A flip-flop output hovering in an undefined state after a setup/hold violation, resolving to 0 or 1 at an unpredictable time
- [ ] Two clock domains that happen to run at the same frequency

> When data changes too close to a clock edge (violating setup or hold), the flop can enter a metastable state and take an unbounded time to settle. It is the core hazard at clock-domain crossings and is mitigated with synchronizers, not eliminated outright.
