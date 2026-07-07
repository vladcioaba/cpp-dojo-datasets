## quiz: In a one-hot encoded FSM with 8 states, how many state flip-flops are used and how many are high at once?
tags: fsm, design
track: fpga

- [ ] 3 flip-flops, 1 high
- [x] 8 flip-flops, exactly 1 high
- [ ] 8 flip-flops, all high
- [ ] 3 flip-flops, all high

> One-hot uses one flip-flop per state with exactly one asserted at a time. It costs more registers than binary encoding (which would use 3 bits for 8 states) but yields simpler, faster next-state and decode logic — a good trade on FPGAs, which have flip-flops to spare.
