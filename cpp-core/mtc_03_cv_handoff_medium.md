## challenge: one-slot channel handoff
tags: concurrency, condition-variable
track: core
difficulty: medium

Build a one-slot channel between a producer and a consumer using a mutex and a condition variable. `send()` blocks until the slot is empty, deposits the value, and signals; `receive()` blocks until the slot is full, takes the value, and signals. The harness sends 1..200 and must receive them in order — nothing lost, nothing duplicated. Waits must be real waits (condition variable), not spin loops.

hint: The waiting side needs `std::unique_lock` (a condition_variable must be able to unlock and relock during wait) — `lock_guard` cannot do that.
hint: Always wait with a predicate: `cv_.wait(lock, [this]{ return !full_; })` in send, `[this]{ return full_; }` in receive. This survives spurious wakeups — a bare wait() does not.
hint: Change `full_` while holding the mutex, then notify. Since one condition_variable serves both "became full" and "became empty", notify_all() is the safe choice.

```cpp
// starter
// One-slot channel: send() blocks while the slot is occupied,
// receive() blocks while it is empty. Values arrive in send order.
class Channel {
public:
    void send(int v) {
        // TODO: wait (predicate loop!) until !full_,
        //       then store v, mark full_, and notify.
    }
    int receive() {
        // TODO: wait (predicate loop!) until full_,
        //       then clear full_, notify, and return the value.
        return -1;
    }
private:
    std::mutex m_;
    std::condition_variable cv_;
    int slot_ = 0;
    bool full_ = false;
};
```

```cpp
class Channel {
public:
    void send(int v) {
        std::unique_lock<std::mutex> lock(m_);
        cv_.wait(lock, [this] { return !full_; });  // sleeps until slot free
        slot_ = v;
        full_ = true;
        cv_.notify_all();                           // wake the receiver
    }
    int receive() {
        std::unique_lock<std::mutex> lock(m_);
        cv_.wait(lock, [this] { return full_; });   // sleeps until slot full
        full_ = false;
        int v = slot_;
        cv_.notify_all();                           // wake the sender
        return v;
    }
private:
    std::mutex m_;
    std::condition_variable cv_;
    int slot_ = 0;
    bool full_ = false;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Channel ch;
    std::thread producer([&ch] {
        for (int i = 1; i <= 200; ++i) ch.send(i);
    });
    long sum = 0;
    for (int i = 1; i <= 200; ++i) {
        int v = ch.receive();
        assert(v == i);            // strict order: nothing lost or duplicated
        sum += v;
    }
    producer.join();
    assert(sum == 200 * 201 / 2);
    std::puts("PASS");
}
```

**Editorial:** The canonical condition-variable rendezvous. `wait(lock, pred)` expands to `while (!pred()) wait(lock);` — the loop is load-bearing: the OS may wake a waiter spuriously, and even a genuine notify only proves the condition *was* true a moment ago. `wait` atomically releases the mutex and sleeps, which closes the lost-wakeup gap — provided the notifier flips `full_` *under the same mutex*. `unique_lock` is required because `wait` must unlock and relock mid-flight. One CV serves both directions here, so `notify_all` is the robust choice; with exactly one thread per side `notify_one` also works, but the moment a second producer appears it can wake the wrong class of waiter — the predicate loop is what keeps even that case correct.
