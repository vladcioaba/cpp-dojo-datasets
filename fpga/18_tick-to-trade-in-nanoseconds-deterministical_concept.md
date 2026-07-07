## fact: Tick-to-trade in nanoseconds, deterministically
tags: hft, latency, determinism
track: fpga

**Tick-to-trade** is the time from a market-data packet arriving to an order leaving the wire. A tuned software stack — even with kernel bypass — lands in the **single-digit microseconds**. An FPGA doing the same path as a circuit lands in **tens to low hundreds of nanoseconds**, often about an order of magnitude faster.

Just as important as the mean is the **determinism**: no OS scheduler, no cache misses, no garbage collection, no interrupts. Every message takes the *same* number of clock cycles. In trading, the latency tail is what gets you picked off, so a predictable 200 ns can beat an average 1 µs that occasionally spikes to 50 µs.
