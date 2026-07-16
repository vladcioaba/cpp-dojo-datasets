## challenge: a counter four threads can trust
tags: concurrency, mutex, threads
track: core
difficulty: easy

Four worker threads will each call `increment()` 25,000 times. Make `Counter` thread-safe with a `std::mutex` so the final value is exactly 100,000 — no lost updates, ever. Keep the public interface unchanged.

hint: `++count_` is three steps — read, add, write — and two threads can interleave them, both writing back the same value.
hint: Add a `std::mutex` member and hold it for the entire read-modify-write. Use `std::lock_guard`, not manual lock()/unlock().
hint: `value()` is const but must also lock — a read racing a write is still a data race. Declare the mutex `mutable`.

```cpp
// starter
// Make Counter safe to call from many threads at once.
class Counter {
public:
    void increment() {
        // TODO: protect the update with a std::mutex so that
        //       concurrent increments are never lost.
        (void)count_;
    }
    long value() const {
        // TODO: reads need the same protection as writes.
        return count_;
    }
private:
    // TODO: add the mutex member (hint: value() is const...)
    long count_ = 0;
};
```

```cpp
class Counter {
public:
    void increment() {
        std::lock_guard<std::mutex> lock(m_);   // unlocks on every exit path
        ++count_;
    }
    long value() const {
        std::lock_guard<std::mutex> lock(m_);   // readers lock too
        return count_;
    }
private:
    mutable std::mutex m_;   // mutable: lockable inside const methods
    long count_ = 0;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Counter c;
    std::vector<std::thread> workers;
    for (int t = 0; t < 4; ++t)
        workers.emplace_back([&c] {
            for (int i = 0; i < 25000; ++i) c.increment();
        });
    for (auto& w : workers) w.join();
    assert(c.value() == 100000);   // 4 threads x 25,000 — nothing lost
    std::puts("PASS");
}
```

**Editorial:** The unprotected `++count_` compiles to load, add, store; when two threads interleave those steps they both write back the same value and one increment vanishes. Holding a `std::mutex` across the whole read-modify-write makes the sequence indivisible, and the unlock/lock pair also publishes the new value to the next thread (release/acquire semantics for free). Two details reviewers look for: `std::lock_guard` instead of manual `lock()`/`unlock()` so an exception can't leave the mutex held, and a `mutable` mutex so the `const` reader path locks too — an unlocked read racing a locked write is still undefined behavior.
