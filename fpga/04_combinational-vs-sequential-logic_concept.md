## fact: Combinational vs sequential logic
tags: hdl, fundamentals
track: fpga

**Combinational logic** is a pure function of its current inputs — LUTs computing an output with no memory. In Verilog it's an `assign` or an `always @(*)` block, and its cost is propagation delay as signals ripple through gates.

**Sequential logic** holds state that updates on a clock edge, built from flip-flops. `always @(posedge clk)` describes registers that capture their inputs once per rising edge and hold them until the next. Real designs alternate: combinational logic computes, a register latches the result, the next stage computes from there.
