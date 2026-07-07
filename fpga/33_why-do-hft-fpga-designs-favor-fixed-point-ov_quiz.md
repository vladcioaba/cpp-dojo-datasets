## quiz: Why do HFT FPGA designs favor fixed-point over floating-point arithmetic?
tags: arithmetic, dsp
track: fpga

- [ ] Floating-point cannot be represented in hardware at all
- [x] Floating-point costs far more LUTs/DSPs and adds pipeline latency; fixed-point maps directly onto DSP slices
- [ ] Fixed-point is strictly more accurate than floating-point in every case
- [ ] FPGAs are unable to perform multiplication

> Float requires exponent handling, normalization, and rounding built from fabric, costing resources and latency per operation. Prices and sizes are integers anyway, so scaled fixed-point runs in a DSP slice in a cycle or two. It's a cost/latency argument, not accuracy — and FPGAs multiply just fine (that's what DSP slices do).
