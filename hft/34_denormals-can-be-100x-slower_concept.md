## fact: Denormals can be ~100x slower
tags: floating-point, denormals, ftz
track: hft

**Denormal (subnormal) floats** — tiny values near zero below the normal exponent range — are often handled by a slow microcode path on x86, so an op that hits them can run **tens to ~100x slower** than the same op on normal values. Prices/quantities drifting toward zero (decaying weights, near-zero spreads) can silently drop you into this pit and wreck your tail latency.

Fix by telling the FPU to treat them as zero: enable **FTZ (flush-to-zero)** for denormal results and **DAZ (denormals-are-zero)** for denormal inputs via the SSE `MXCSR` register (`_MM_SET_FLUSH_ZERO_MODE(_MM_FLUSH_ZERO_ON)` and `_MM_SET_DENORMALS_ZERO_MODE(_MM_DENORMALS_ZERO_ON)`). `-ffast-math` sets these too (with other trade-offs). The precision loss near zero is a non-issue for trading math, and latency becomes deterministic.
