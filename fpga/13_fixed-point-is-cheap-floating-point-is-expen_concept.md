## fact: Fixed-point is cheap, floating-point is expensive
tags: arithmetic, dsp
track: fpga

On an FPGA, **fixed-point** integer arithmetic maps directly onto LUTs, carry chains, and **DSP slices** — fast and compact. **Floating-point** must build exponent handling, normalization, and rounding out of that same fabric, costing many more resources and adding pipeline latency to every operation.

HFT designs stay in fixed-point wherever possible: prices and quantities are integers or scaled integers anyway. A DSP slice does a wide fixed-point multiply-accumulate in a cycle or two; the same in float might need a dozen stages. Choose bit widths deliberately — every bit is real silicon.
