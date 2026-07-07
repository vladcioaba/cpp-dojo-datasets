## fact: Blocking vs lock-free vs wait-free
tags: concurrency, lock-free, wait-free
track: hft

These describe **progress guarantees**, not merely "no mutex":

- **Blocking**: a thread can be stalled indefinitely by another. If a mutex holder is descheduled, everyone waits — priority inversion and unbounded latency, which is what HFT fears.
- **Lock-free**: the system as a whole always progresses — at least one thread completes in a bounded number of steps — but an individual thread can starve (a CAS loop that keeps retrying).
- **Wait-free**: *every* thread completes in a bounded number of *its own* steps regardless of others. Strongest guarantee, hardest to build; gives the tightest worst-case (tail) latency.

The HFT appeal is bounded tail latency and immunity to a descheduled thread stalling the pipeline — not raw throughput. Lock-free is not automatically faster than a well-used mutex; it's about the worst case, not the average.
