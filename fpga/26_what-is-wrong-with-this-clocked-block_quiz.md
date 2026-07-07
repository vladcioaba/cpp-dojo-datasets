## quiz: What is wrong with this clocked block?
tags: hdl, verilog, gotcha
track: fpga

```verilog
always @(posedge clk) begin
    b = a;   // blocking
    c = b;   // blocking
end
```

- [ ] Nothing — blocking assignments are correct in clocked blocks
- [x] Sequential logic should use non-blocking `<=`; with blocking `=`, `c` sees the new `b` immediately, collapsing an intended 2-stage path into one and risking sim/synth mismatch
- [ ] It creates a combinational feedback loop
- [ ] `b` and `c` will hold values from two different clocks

> In a `posedge clk` block, use non-blocking `<=` so every right-hand side samples the *old* values and all registers update together — modeling real flip-flops. With blocking `=`, `c = b` uses the `b` just assigned this line, so both `b` and `c` end up equal to `a` and the intended two-register delay disappears. Blocking `=` belongs in combinational (`always @(*)`) blocks.
