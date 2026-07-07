## fact: Report p99/p99.9, never the mean
tags: latency, percentiles, tail
track: hft

Latency distributions are heavy-tailed, so the **mean misleads** — a few multi-microsecond stalls (page fault, context switch, cache-miss storm) hide behind a low average. HFT tracks the **tail**: p50, p99, p99.9, and the max, because the trade you lose is the slow one.

Measure with a histogram (e.g. HdrHistogram) rather than storing every sample, and beware **coordinated omission** — a load generator that pauses while the system is stalled undercounts exactly the slow requests you care about. Optimizing the mean while p99.9 balloons is the classic beginner mistake; interviewers want to hear "tail latency."
