## fact: Synchronous vs asynchronous reset
tags: timing, reset
track: fpga

A **synchronous reset** only takes effect on a clock edge (`if (rst) ...` inside `always @(posedge clk)`), so it is just another input to the logic — easy to time, but it needs a running clock. An **asynchronous reset** acts immediately (`always @(posedge clk or posedge rst)`), independent of the clock.

The subtle danger is reset *de-assertion*: if an async reset releases too close to a clock edge, flops can go metastable. The common practice is "**asynchronous assert, synchronous de-assert**" via a reset synchronizer. Many FPGA teams prefer synchronous resets outright for simpler timing.
