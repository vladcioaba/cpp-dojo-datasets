## challenge: lock-free ticket counter
tags: concurrency, atomics
track: core
difficulty: easy

Same job as the mutex counter — four threads, 25,000 `take()` calls each, exact total of 100,000 — but this time the shared state is a single integer, so a mutex is overkill. Make `TicketCounter` lock-free with `std::atomic`.

hint: Change the member's type: `std::atomic<long>`. The counter is a single independent scalar — exactly what atomics are for.
hint: The atomic read-modify-write for "add and return the old value" is `fetch_add(1)`. Plain `issued_ = issued_ + 1` on an atomic is TWO operations and still loses updates.
hint: A standalone counter needs no ordering with other data — `std::memory_order_relaxed` is correct and fastest. (The default `seq_cst` also passes.)

```cpp
// starter
// Lock-free counter: no mutex allowed — use std::atomic so that
// concurrent take() calls are indivisible.
class TicketCounter {
public:
    void take() {
        // TODO: increment atomically (one fetch_add, not load-then-store)
    }
    long issued() const {
        return 0;   // TODO: load the current value
    }
private:
    long issued_ = 0;   // TODO: make this std::atomic<long>
};
```

```cpp
class TicketCounter {
public:
    void take() {
        issued_.fetch_add(1, std::memory_order_relaxed);  // one indivisible RMW
    }
    long issued() const {
        return issued_.load(std::memory_order_relaxed);
    }
private:
    std::atomic<long> issued_{0};
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    TicketCounter c;
    std::vector<std::thread> workers;
    for (int t = 0; t < 4; ++t)
        workers.emplace_back([&c] {
            for (int i = 0; i < 25000; ++i) c.take();
        });
    for (auto& w : workers) w.join();
    assert(c.issued() == 100000);   // atomic RMW: nothing lost, no mutex
    std::puts("PASS");
}
```

**Editorial:** `fetch_add(1)` performs the read-modify-write as one indivisible hardware operation (`lock xadd` on x86, `ldadd` on ARM), so interleaving is impossible — the correctness of the mutex version at a fraction of the cost. The trap the exercise plants: writing `issued_ = issued_ + 1` even on an `atomic<long>` is an atomic load followed by a separate atomic store — no lost *bits*, but lost *increments* all the same. `memory_order_relaxed` is enough here because the counter carries no dependent data; the joins at the end give the main thread a happens-before edge to read the final total. When an invariant spans two variables, though, atomics stop composing — that's mutex territory.
