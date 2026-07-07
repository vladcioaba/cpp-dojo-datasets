## fact: Setup time, hold time, and metastability
tags: timing, fundamentals
track: fpga

A flip-flop only captures data reliably if its input is stable in a window around the clock edge. **Setup time** is how long data must be steady *before* the edge; **hold time** is how long it must stay steady *after*. Violate either and the flop can go **metastable** — its output hovers in an undefined state and resolves at an unpredictable time.

Setup violations are fixed by slowing the clock or shortening the path (more pipelining). Hold violations are nastier: they don't improve with a slower clock because the offending path is too *short* relative to clock skew, so the place-and-route tool must add delay.
