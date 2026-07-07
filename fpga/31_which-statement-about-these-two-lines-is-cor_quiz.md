## quiz: Which statement about these two lines is correct?
tags: hdl, fundamentals
track: fpga

```verilog
assign y = a & b;
always @(posedge clk) q <= d;
```

- [x] `y` is combinational (tracks a and b continuously); `q` is sequential (updates only on the clock edge)
- [ ] Both are sequential because they sit in the same module
- [ ] `y` is sequential because `assign` is a stateful construct
- [ ] `q` is combinational because it has no reset

> `assign` describes combinational logic with no memory — its output follows its inputs continuously. The `always @(posedge clk)` block infers a flip-flop that captures `d` once per rising edge. Whether a reset exists has nothing to do with combinational vs sequential.
