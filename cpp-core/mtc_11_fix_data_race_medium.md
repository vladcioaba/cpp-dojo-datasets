## challenge: fix: the hit counter comes up short
tags: concurrency, data-race, debugging, code-review
track: core
difficulty: medium

This code review found a bug: the nightly hit totals from `HitCounter` are always lower than the load balancer's request count — updates are being lost. The `audit_hook()` call must stay exactly where it is in the flow (compliance logs between the read and the write). Find and fix it — keep the public interface.

hint: hit() is a read-modify-write in three visible steps: read hits_, run the hook, write back. What happens when two threads both read the same snapshot before either writes?
hint: Nothing stops threads from interleaving inside hit() — this is a textbook data race (concurrent access, at least one write, no synchronization). The whole read-hook-write sequence must be one critical section.
hint: Add a std::mutex member; take a std::lock_guard for the entire body of hit() — including the audit_hook() call — and lock in total() too (a racing read is still UB). Mark the mutex mutable for the const method.

```cpp
// starter
// Counts requests across worker threads. audit_hook() must run
// between reading the old count and writing the new one.
class HitCounter {
public:
    void hit() {
        long snapshot = hits_;      // read
        audit_hook();               // compliance hook — keep this call here
        hits_ = snapshot + 1;       // write back
    }
    long total() const { return hits_; }
private:
    long hits_ = 0;
};
```

```cpp
// Counts requests across worker threads. audit_hook() must run
// between reading the old count and writing the new one.
class HitCounter {
public:
    void hit() {
        std::lock_guard<std::mutex> lock(m_);  // one thread at a time
        long snapshot = hits_;      // read
        audit_hook();               // still between read and write — now safe
        hits_ = snapshot + 1;       // write back
    }
    long total() const {
        std::lock_guard<std::mutex> lock(m_);
        return hits_;
    }
private:
    mutable std::mutex m_;
    long hits_ = 0;
};
```

```cpp
// harness
#include <bits/stdc++.h>
void audit_hook() { std::this_thread::yield(); }  // other threads run here
//__USER__
int main() {
    HitCounter c;
    std::vector<std::thread> workers;
    for (int t = 0; t < 4; ++t)
        workers.emplace_back([&c] {
            for (int i = 0; i < 2000; ++i) c.hit();
        });
    for (auto& w : workers) w.join();
    assert(c.total() == 8000);   // 4 x 2000 — every hit counted
    std::puts("PASS");
}
```

**Editorial:** The race window is spelled out in the code: thread A reads `hits_ == 41`, the hook yields the CPU, threads B, C, D each read the *same* 41 and write 42, then A finally writes its own 42 — three hits gone. Any unsynchronized read-modify-write has this window; the hook only widens it from nanoseconds to a certainty. The fix is not to move the hook but to make the whole sequence a critical section: one `std::mutex`, one `std::lock_guard` spanning read, hook, and write — and the same lock in `total()`, because a read racing a write is undefined behavior even when only the write "changes" anything. If the hook constraint ever disappeared, `std::atomic<long>::fetch_add` would do the job lock-free.
