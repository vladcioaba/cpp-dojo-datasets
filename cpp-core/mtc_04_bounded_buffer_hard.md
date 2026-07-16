## challenge: bounded buffer under pressure
tags: concurrency, condition-variable, producer-consumer
track: core
difficulty: hard

The full producer/consumer queue: a FIFO with a hard capacity of 4 slots. `push()` blocks while the buffer is full (backpressure — producers must not outrun the consumer's memory), `pop()` blocks while it is empty. Two producer threads push 1,000 items each while the main thread pops all 2,000; every element must make it through exactly once. Use a mutex plus *two* condition variables — one per direction.

hint: Two waits, two conditions: push waits for `queue_.size() < capacity_` and notifies not_empty_ after inserting; pop waits for `!queue_.empty()` and notifies not_full_ after removing.
hint: Both waits need std::unique_lock and a predicate lambda — with multiple producers blocked at once, spurious and stolen wakeups WILL happen; the predicate loop re-checks after every wake.
hint: Why two condition variables? With one shared cv and notify_one, a producer's notify can wake another producer (still full, goes back to sleep) instead of the consumer — a stall. Separate cvs make every notify land on the side that can act.

```cpp
// starter
// Fixed-capacity FIFO shared by producers and consumers.
// push() blocks while full; pop() blocks while empty.
class BoundedBuffer {
public:
    explicit BoundedBuffer(std::size_t capacity) : capacity_(capacity) {}
    void push(int v) {
        // TODO: wait on not_full_ until queue_.size() < capacity_,
        //       push_back, notify not_empty_.
    }
    int pop() {
        // TODO: wait on not_empty_ until !queue_.empty(),
        //       pop_front, notify not_full_, return the value.
        return 0;
    }
private:
    std::size_t capacity_;
    std::deque<int> queue_;
    std::mutex m_;
    std::condition_variable not_full_;
    std::condition_variable not_empty_;
};
```

```cpp
class BoundedBuffer {
public:
    explicit BoundedBuffer(std::size_t capacity) : capacity_(capacity) {}
    void push(int v) {
        std::unique_lock<std::mutex> lock(m_);
        not_full_.wait(lock, [this] { return queue_.size() < capacity_; });
        queue_.push_back(v);
        not_empty_.notify_one();     // exactly the side that can proceed
    }
    int pop() {
        std::unique_lock<std::mutex> lock(m_);
        not_empty_.wait(lock, [this] { return !queue_.empty(); });
        int v = queue_.front();
        queue_.pop_front();
        not_full_.notify_one();      // a producer slot just opened
        return v;
    }
private:
    std::size_t capacity_;
    std::deque<int> queue_;
    std::mutex m_;
    std::condition_variable not_full_;
    std::condition_variable not_empty_;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    BoundedBuffer buf(4);   // tiny capacity: producers must block on backpressure
    auto produce = [&buf](int base) {
        for (int i = 1; i <= 1000; ++i) buf.push(base + i);
    };
    std::thread p1(produce, 0);        // pushes 1..1000
    std::thread p2(produce, 100000);   // pushes 100001..101000
    long long sum = 0;
    for (int i = 0; i < 2000; ++i) sum += buf.pop();   // consumer: main thread
    p1.join();
    p2.join();
    long long expected = 2 * (1000LL * 1001 / 2) + 1000LL * 100000;
    assert(sum == expected);           // every item through, exactly once
    std::puts("PASS");
}
```

**Editorial:** This is the pattern behind every work queue and pipeline stage. Capacity 4 against 2,000 items guarantees the interesting states actually occur: producers pile up on `not_full_`, the consumer drains, everyone hands off through the same mutex. The two-CV design is not cosmetic — with a single CV and `notify_one`, a wakeup can land on a waiter from the *same* side (producer wakes producer while the buffer is still full), which stalls until a spurious wake rescues you; splitting the channels makes each notify meaningful. Note that both notifies are issued while the lock is held — correct, and simplest; dropping the lock first is a micro-optimization that must not be attempted before the predicate discipline is second nature. The distinct value ranges (1..1000 vs 100001..101000) make the checksum sensitive to any lost or duplicated element from either producer.
