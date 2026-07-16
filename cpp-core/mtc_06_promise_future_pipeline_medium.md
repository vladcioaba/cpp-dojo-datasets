## challenge: promise/future relay
tags: concurrency, futures, promise
track: core
difficulty: medium

Wire a two-stage pipeline by hand — no `std::async` this time. Stage 1 runs in its own thread and squares the seed; stage 2 runs in another thread, waits for stage 1's result, and adds 100. Each handoff is a `std::promise`/`std::future` pair: the promise is the writing end, the future the reading end. `pipeline()` returns the final value.

hint: `std::promise<int> p1; std::future<int> f1 = p1.get_future();` — create the pair BEFORE launching the thread, move/capture the promise into the producer, keep the future on the consuming side.
hint: Stage 2's thread body is just `p2.set_value(f1.get() + 100);` — future::get() blocks until stage 1 calls set_value, then hands over the value. get() may be called only once per future.
hint: Join both threads before returning. (set_value also propagates through a stored exception if you use set_exception — the same channel carries errors.)

```cpp
// starter
// Stage 1 (own thread): seed * seed  --promise/future-->
// Stage 2 (own thread): result + 100 --promise/future--> caller.
int pipeline(int seed) {
    // TODO:
    //   promise<int> p1 + future f1; thread A: p1.set_value(seed * seed)
    //   promise<int> p2 + future f2; thread B: p2.set_value(f1.get() + 100)
    //   join both, return f2.get()
    return 0;
}
```

```cpp
int pipeline(int seed) {
    std::promise<int> p1;
    std::future<int> f1 = p1.get_future();
    std::thread stage1([&p1, seed] {
        p1.set_value(seed * seed);           // fulfil the first promise
    });

    std::promise<int> p2;
    std::future<int> f2 = p2.get_future();
    std::thread stage2([&p2, &f1] {
        p2.set_value(f1.get() + 100);        // blocks until stage 1 delivers
    });

    stage1.join();
    stage2.join();
    return f2.get();
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(pipeline(0) == 100);     // 0*0 + 100
    assert(pipeline(5) == 125);     // 25 + 100
    assert(pipeline(-4) == 116);    // 16 + 100
    assert(pipeline(12) == 244);    // 144 + 100
    std::puts("PASS");
}
```

**Editorial:** `std::promise`/`std::future` is the raw one-shot channel underneath `std::async`: `set_value` on one side, a blocking `get()` on the other, with the synchronization (the value written before `set_value` happens-before the return of `get()`) built in — no mutex, no condition variable, no flag. The order of operations matters: create the pair, *then* launch the thread, so the future already exists when the producer runs. Two classic slips this exercise flushes out: calling `get()` twice on the same future (it's a one-shot — the second call is UB on a moved-from state), and forgetting that a destroyed promise with no value set poisons its future with `broken_promise`. For error paths, `set_exception` uses the same channel — the exception rethrows out of `get()`.
