## challenge: Atomic max via CAS loop
tags: atomics, cas-loop, compare-exchange
track: hft
difficulty: medium

`std::atomic` gives you `fetch_add` and `fetch_and`, but there is no `fetch_max`. To track a high-water mark (peak queue depth, max observed latency) across threads you must build it yourself with a **compare-exchange loop**. Implement `void update(long x)` that atomically sets the stored value to `max(current, x)`, and `long get() const`. The pattern: load the current value; if `x` isn't larger, stop; otherwise CAS current→x, and on failure re-read (the CAS refreshes your expected) and re-test. Start the value at `LONG_MIN`. Single-threaded correctness; the harness feeds a sequence and checks the running maximum.

Constraints: no locks. `update` must be a lock-free CAS loop, not a plain load-then-store.

Example: feeding 3,1,4,1,5,9,2,6 leaves the max at 9; a later `update(-100)` is ignored, `update(1000)` raises it to 1000.

hint: If `x <= current` there is nothing to do — bail out before touching the atomic. That also makes the common case a single load.
hint: `compare_exchange_weak(cur, x)` writes the freshly-observed value back into `cur` on failure, so your loop re-tests `x > cur` with no extra load.
hint: The value orders no other memory, so both the success and failure legs can be `relaxed`.

```cpp
// starter
#include <atomic>
#include <climits>
struct AtomicMax {
    std::atomic<long> val_{LONG_MIN};
    void update(long x);   // val_ = max(val_, x), atomically
    long get() const;
};
```

```cpp
void update(long x) {
    long cur = val_.load(std::memory_order_relaxed);
    while (x > cur && !val_.compare_exchange_weak(cur, x,
               std::memory_order_relaxed, std::memory_order_relaxed)) {
        // CAS failed: cur now holds the latest value; loop re-checks x > cur
    }
}
long get() const { return val_.load(std::memory_order_relaxed); }
```

```cpp
// harness
#include <cstdio>
#include <atomic>
#include <climits>
struct AtomicMax {
    std::atomic<long> val_{LONG_MIN};
    //__USER__
};
int main() {
    AtomicMax m;
    long data[] = {3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5};
    long best = LONG_MIN;
    for (long v : data) {
        m.update(v);
        if (v > best) best = v;
        if (m.get() != best) { std::puts("running max wrong"); return 1; }
    }
    if (m.get() != 9) { std::puts("final max wrong"); return 1; }
    m.update(-100);
    if (m.get() != 9) { std::puts("smaller must be ignored"); return 1; }
    m.update(1000);
    if (m.get() != 1000) { std::puts("larger must apply"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Any read-modify-write the hardware doesn't provide natively is synthesized from a CAS loop, and `fetch_max` is the textbook case. The invariant that makes it correct under concurrency: you only ever install `x` if the value you're overwriting is still the one you decided was smaller than `x`. If another thread slips a larger value in between your load and your CAS, the compare fails, `compare_exchange_weak` hands you the new current value, and you re-evaluate `x > cur` — which may now be false, so you correctly drop out without clobbering the larger value. The early `x > cur` guard means non-improving updates cost a single relaxed load with zero writes, which matters when the max rarely moves. Minimal-correct ordering is `relaxed` on every leg: like a plain counter, the maximum publishes no other memory, so it needs atomicity but no fences. Use `_weak` in the loop (cheaper on ARM/POWER LL-SC, and a spurious failure just spins once more); reach for `_strong` only when there is no surrounding loop to absorb a spurious failure.
