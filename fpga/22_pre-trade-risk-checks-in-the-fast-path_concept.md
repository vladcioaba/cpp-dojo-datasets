## fact: Pre-trade risk checks in the fast path
tags: hft, risk
track: fpga

Regulation and prudence require every outbound order to pass **pre-trade risk** limits — max order size, price bands, per-symbol and aggregate position caps, message-rate throttles. In a software system these checks add microseconds; in hardware they are a handful of comparators the order flows through in a couple of clock cycles.

Putting the risk gate **in the FPGA** makes it unconditional and deterministic — no order can escape to the exchange without passing it, and it costs almost no latency. It is one of the cases where hardware buys you both safety and speed instead of trading one for the other.
