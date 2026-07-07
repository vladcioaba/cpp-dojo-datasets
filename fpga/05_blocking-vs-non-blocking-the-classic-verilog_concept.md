## fact: Blocking `=` vs non-blocking `<=` — the classic Verilog trap
tags: hdl, verilog, gotcha
track: fpga

The rule that prevents a whole class of bugs: use **non-blocking `<=` in clocked (sequential) blocks**, and **blocking `=` in combinational blocks**. Non-blocking assignments all sample their right-hand sides first, then update together at the edge — exactly how real flip-flops behave.

```verilog
// Sequential: a 2-stage shift register. Non-blocking is required.
always @(posedge clk) begin
    q1 <= d;
    q2 <= q1;   // sees the OLD q1, so this is a true 2-stage delay
end

// Combinational: blocking, so each line sees the previous result.
always @(*) begin
    x = a & b;
    y = x | c;  // uses the x computed on the line above
end
```

Swap them and the shift register collapses into a single stage, and you can get a design that simulates one way but synthesizes another.
