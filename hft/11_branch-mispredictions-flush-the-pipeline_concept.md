## fact: Branch mispredictions flush the pipeline
tags: branch-prediction, pipeline, branchless
track: hft

Deep out-of-order pipelines speculate past every branch. A correctly predicted branch is nearly free; a **misprediction flushes the pipeline for ~15-20 cycles** on modern x86. Predictable branches (a bounds check that almost never fires) are cheap; data-dependent, ~50/50 branches are the killers.

`[[likely]]`/`[[unlikely]]` (C++20) hint the compiler which side is hot to improve code layout, but they don't override the hardware predictor and are easy to misuse — profile first.

When a branch is inherently unpredictable, go **branchless** so there's nothing to mispredict — the compiler lowers a ternary to a conditional move (`cmov`), or you mask arithmetically:

```cpp
int m = (a > b) ? a : b;             // compiler emits cmov — no branch
std::uint64_t mask = -std::uint64_t(cond); // cond is 0/1 -> 0x0 or 0xFFFF...F
sum += x & mask;                     // adds x iff cond, no branch
```

Branchless isn't always a win — a `cmov` carries a data dependency and can't be speculated past — so measure.
