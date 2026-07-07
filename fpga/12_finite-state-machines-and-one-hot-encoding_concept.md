## fact: Finite state machines and one-hot encoding
tags: fsm, design
track: fpga

Whenever hardware must do things *in sequence* — a handshake, a protocol, a multi-step decode — you build a **finite state machine**: a state register plus combinational next-state and output logic. FSMs are how you reintroduce ordering into an inherently parallel fabric.

**One-hot encoding** gives each state its own flip-flop, with exactly one asserted at a time. It burns more registers than binary encoding (which would use `log2(N)` bits for N states) but makes next-state and decode logic trivial and fast — and FPGAs have flip-flops in abundance, so one-hot is often the default for speed.
