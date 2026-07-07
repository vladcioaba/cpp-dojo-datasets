## quiz: Denormals (subnormals) and low-latency floating point
tags: floating-point, denormals, ftz
track: hft

- [ ] Denormals speed up math because they use fewer significant bits
- [x] Producing or consuming subnormal results can be 10–100x slower via a microcode path; enabling FTZ/DAZ flushes them to zero to keep latency deterministic
- [ ] Denormals affect only correctness, never performance
- [ ] FTZ increases precision near zero

> Subnormal (denormal) floats fill the gap between the smallest normal number and zero, but on x86 the hardware often handles them through a slow microcode assist, producing large, unpredictable latency spikes when your data drifts toward zero. Setting FTZ (flush results to zero) and DAZ (treat inputs as zero) in the MXCSR trades a little precision near zero for constant, fast behavior — standard practice in trading code.
