## challenge: publish the payload — acquire/release
tags: concurrency, memory-model, atomics
track: core
difficulty: hard

The flag-and-data pair, straight from the interview canon. One writer thread calls `put(value)`; one reader thread calls `take()`, which must spin until the value is published and then return it. The payload itself is a plain `int` — only the flag is atomic — so the flag's memory orderings must carry the synchronization: everything written before the release store must be visible after the acquire load that observes it. Choose the orderings deliberately; be ready to say why `relaxed` would be wrong.

hint: put() in two steps: write payload_ first, THEN `ready_.store(true, std::memory_order_release)`. The release makes every prior write part of the publication.
hint: take() spins: `while (!ready_.load(std::memory_order_acquire)) std::this_thread::yield();` then returns payload_. The acquire load that sees true synchronizes-with the release store — creating the happens-before edge that makes reading the non-atomic payload_ legal.
hint: With relaxed on both sides the flag itself is still atomic, but NOTHING orders payload_ against it — the reader can see ready==true yet a stale payload, and the race on payload_ is UB. seq_cst also passes (it is release/acquire plus more); relaxed does not.

```cpp
// starter
// One-shot mailbox: one writer calls put(), one reader calls take().
// payload_ is deliberately NOT atomic — the flag must publish it.
class Mailbox {
public:
    void put(int value) {
        payload_ = value;
        // TODO: publish — store true into ready_ with the ordering
        //       that makes the payload_ write visible to the reader.
    }
    int take() {
        // TODO: spin (with std::this_thread::yield()) until ready_,
        //       loading with the matching ordering, then return payload_.
        return -1;
    }
private:
    int payload_ = 0;
    std::atomic<bool> ready_{false};
};
```

```cpp
class Mailbox {
public:
    void put(int value) {
        payload_ = value;                                // A: write the data
        ready_.store(true, std::memory_order_release);   // B: publish A
    }
    int take() {
        while (!ready_.load(std::memory_order_acquire))  // C: observes B
            std::this_thread::yield();
        return payload_;                                 // D: A visible — safe
    }
private:
    int payload_ = 0;
    std::atomic<bool> ready_{false};
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    for (int round = 0; round < 300; ++round) {
        Mailbox box;
        int expected = round * 7 + 1;
        std::thread writer([&box, expected] { box.put(expected); });
        int got = box.take();
        writer.join();
        assert(got == expected);   // flag seen => payload must be too
    }
    std::puts("PASS");
}
```

**Editorial:** The release/acquire pair is the C++ memory model in one idiom. `store(release)` says "everything I wrote before this is part of the message"; the `load(acquire)` that reads that value *synchronizes-with* it, establishing happens-before from the payload write (A) to the payload read (D) — which is precisely what makes the non-atomic `payload_` access race-free. Demote both to `relaxed` and the flag stays atomic but orders nothing: compiler and CPU may reorder A past B or D before C, and the reader can observe `ready == true` with a stale payload — undefined behavior that x86 hardware will often hide and ARM will happily expose. `seq_cst` (the default) is correct here too, since it includes acquire/release; the point of naming the orders is to show you know *which* guarantee the code actually relies on. Mutexes give the same edge implicitly: unlock releases, lock acquires.
