## fact: The clock, Fmax, and the critical path
tags: timing, fundamentals
track: fpga

Synchronous designs march to a **clock**. Between any two registers sits combinational logic with some propagation delay; the **critical path** is the slowest such path in the entire design. The clock period must exceed that delay (plus setup time), so the critical path sets the maximum clock frequency, **Fmax** — one long path can cap the whole chip's speed.

**Timing closure** is the iterative work of getting a design to meet its target clock: restructuring logic, adding pipeline registers, guiding placement, until the tools report no negative slack.
