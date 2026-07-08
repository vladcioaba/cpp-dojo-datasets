## challenge: Spinlock
tags: lock-free, spinlock, atomics
track: hft
difficulty: easy

The simplest mutual-exclusion primitive in a hot path: a **test-and-set spinlock** over a single `std::atomic<bool>`. Implement `void lock()` (busy-wait until acquired), `bool try_lock()` (grab the lock if free, otherwise return `false` without blocking), and `void unlock()` (release it). Acquisition must *atomically* observe "was free, now held" so two callers can never both win. Single-threaded correctness only here — the harness checks the lock/try_lock/unlock state machine; the real thing pairs `acquire` on the take with `release` on the drop.

Constraints: no `std::mutex`, no OS calls — only `std::atomic<bool>` and its operations. `try_lock()` must never spin.

Example: on a fresh lock `try_lock()` returns `true`; a second `try_lock()` (while held) returns `false`; after `unlock()`, `try_lock()` returns `true` again.

hint: A plain "read then write" is two steps and can race. `compare_exchange` fuses them: swap `false`→`true` and tell you whether the swap happened.
hint: `try_lock()` is exactly one `compare_exchange_strong(expected=false, true)` — success means you acquired it.
hint: The take needs `memory_order_acquire`; the release (`unlock`) needs `memory_order_release`, so work inside the critical section can't leak out.

```cpp
// starter
#include <atomic>
struct SpinLock {
    std::atomic<bool> locked_{false};
    void lock();
    bool try_lock();
    void unlock();
};
```

```cpp
void lock() {
    bool expected = false;
    while (!locked_.compare_exchange_weak(expected, true,
               std::memory_order_acquire, std::memory_order_relaxed)) {
        expected = false;   // CAS wrote the observed value back into expected
    }
}
bool try_lock() {
    bool expected = false;
    return locked_.compare_exchange_strong(expected, true,
               std::memory_order_acquire, std::memory_order_relaxed);
}
void unlock() {
    locked_.store(false, std::memory_order_release);
}
```

```cpp
// harness
#include <cstdio>
#include <atomic>
struct SpinLock {
    std::atomic<bool> locked_{false};
    //__USER__
};
int main() {
    SpinLock m;
    if (!m.try_lock()) { std::puts("try_lock on free must succeed"); return 1; }
    if (m.try_lock())  { std::puts("try_lock on held must fail"); return 1; }
    m.unlock();
    if (!m.try_lock()) { std::puts("try_lock after unlock must succeed"); return 1; }
    m.unlock();
    m.lock();
    if (m.try_lock())  { std::puts("lock() must hold the lock"); return 1; }
    m.unlock();
    for (int i = 0; i < 1000; ++i) {
        m.lock();
        if (m.try_lock()) { std::puts("held after lock"); return 1; }
        m.unlock();
    }
    std::puts("PASS");
}
```

**Editorial:** The whole primitive rests on one indivisible action: `compare_exchange` from `false` to `true`. Read-modify-write in a single instruction is what makes "observe free *and* claim it" impossible to interleave, so at most one thread ever sees the transition. `lock()` retries that CAS (use the `_weak` form in a loop — it can fail spuriously but is cheaper on LL/SC machines); `try_lock()` uses `_strong` and returns the outcome. The minimal-correct orders: the successful take is `acquire` and `unlock` is `release`, forming a release/acquire pair so everything a thread wrote before releasing is visible to the next thread that acquires — the failure path of the CAS only needs `relaxed`. Cost is O(1) per operation but unbounded spinning under contention; production versions add a PAUSE/backoff and often a load-before-CAS to avoid hammering the cache line.
