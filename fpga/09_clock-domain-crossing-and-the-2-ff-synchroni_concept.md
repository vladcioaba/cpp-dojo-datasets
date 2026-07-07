## fact: Clock-domain crossing and the 2-FF synchronizer
tags: cdc, timing, gotcha
track: fpga

When a signal passes from one clock domain to another, the receiving flop can sample it mid-transition and go **metastable**. For a **single-bit** signal the standard fix is a **two-flip-flop synchronizer**: two registers in series in the destination domain. The first may go metastable but almost always settles before the second samples it, driving the failure rate astronomically low.

```verilog
always @(posedge clk_dst) begin
    sync0 <= async_in;   // may be metastable
    sync1 <= sync0;      // settled by now; safe to use downstream
end
```

A 2-FF synchronizer only works for one bit at a time. **Multi-bit** buses need Gray coding (for counters), a request/acknowledge handshake, or an asynchronous FIFO — otherwise the bits arrive skewed and you latch a value that never actually existed.
