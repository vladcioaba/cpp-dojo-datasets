## quiz: You need a 4096-entry lookup table read once per cycle. Which resource fits best?
tags: memory, resources
track: fpga

- [ ] Flip-flops (registers)
- [x] BRAM (block RAM)
- [ ] A single LUT
- [ ] DSP slices

> Thousands of entries belong in dedicated BRAM, which is sized for kilobits and up. Holding 4096 entries in flip-flops would burn enormous register and routing resources; a single LUT stores only a tiny truth table; DSP slices do arithmetic, not general storage. (BRAM is dual-port, so up to two reads per cycle.)
