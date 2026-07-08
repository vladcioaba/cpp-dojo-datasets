## challenge: Relaxed atomic counter
tags: atomics, counter, relaxed
track: hft
difficulty: easy

A shared statistics counter (messages processed, packets dropped) that many threads bump but whose *value* nobody uses for synchronization. Wrap a `std::atomic<uint64_t>` and implement `void inc()` (add one), `uint64_t add(uint64_t k)` (add `k`, return the value **before** the add — the fetch-add semantic), and `uint64_t get() const`. Because the count doesn't order any other memory, every operation should use the cheapest correct order: `memory_order_relaxed`. Correctness of the tally is guaranteed by atomicity of `fetch_add` alone.

Constraints: use `fetch_add`, not a load-then-store (that would drop concurrent increments). All operations `relaxed`.

Example: after 100000 `inc()` calls, `get() == 100000`. `add(5)` on a counter at 100000 returns `100000` and leaves it at `100005`.

hint: `n += 1` on an atomic is a load, an add, and a store — three steps that can interleave and lose counts. `fetch_add` is one indivisible RMW.
hint: `fetch_add` returns the *previous* value, which is exactly what `add` must return.
hint: A counter read by no synchronizing thread needs no ordering — `relaxed` still guarantees each RMW is atomic and the total is exact.

```cpp
// starter
#include <atomic>
#include <cstdint>
struct Counter {
    std::atomic<std::uint64_t> n_{0};
    void inc();
    std::uint64_t add(std::uint64_t k);   // returns previous value
    std::uint64_t get() const;
};
```

```cpp
void inc() { n_.fetch_add(1, std::memory_order_relaxed); }
std::uint64_t add(std::uint64_t k) { return n_.fetch_add(k, std::memory_order_relaxed); }
std::uint64_t get() const { return n_.load(std::memory_order_relaxed); }
```

```cpp
// harness
#include <cstdio>
#include <atomic>
#include <cstdint>
struct Counter {
    std::atomic<std::uint64_t> n_{0};
    //__USER__
};
int main() {
    Counter c;
    if (c.get() != 0) { std::puts("init must be 0"); return 1; }
    for (int i = 0; i < 100000; ++i) c.inc();
    if (c.get() != 100000) { std::puts("inc count wrong"); return 1; }
    std::uint64_t prev = c.add(5);
    if (prev != 100000) { std::puts("add must return previous"); return 1; }
    if (c.get() != 100005) { std::puts("add not applied"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The correctness of a concurrent counter has *nothing* to do with memory ordering and *everything* to do with atomicity. `fetch_add` is a single read-modify-write the hardware performs indivisibly (`lock xadd` on x86), so N threads each adding once always yields exactly N — no lost updates. Since the counter value never publishes other data (no reader says "the count is 5, therefore buffer X is ready"), it participates in no happens-before relationship, and `relaxed` — which drops all fences and lets the compiler/CPU reorder surrounding code freely — is the minimal-correct order. That's the point of the drill: reach for `relaxed` when you want an atomic *number*, and reserve `acquire`/`release` for when the number *gates* access to other memory. `fetch_add` returning the prior value also gives you a free unique-ticket generator (see the ticket lock).
