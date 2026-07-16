## challenge: split the sum with std::async
tags: concurrency, async, futures
track: core
difficulty: easy

Implement `parallelSum`: sum a vector by farming the first half out to `std::async` while the calling thread sums the second half, then combine the two results. Mind the launch policy — "async by default" is not what the default does. Handle odd lengths and the empty vector.

hint: Split at `v.begin() + v.size() / 2`. `std::accumulate(first, last, 0LL)` sums a range — the `0LL` keeps the arithmetic in `long long`.
hint: Pass `std::launch::async` explicitly. The default policy may choose `deferred`, which runs the work lazily on YOUR thread at get() — legal, but not parallel.
hint: Launch the async task FIRST, then sum the other half on the current thread, then `lower.get() + upper`. Calling get() before doing your own half serializes everything.

```cpp
// starter
// Sum v using two concurrent halves: one via std::async, one on the
// calling thread. Must be correct for odd sizes and the empty vector.
long long parallelSum(const std::vector<int>& v) {
    // TODO:
    //   auto mid = v.begin() + v.size() / 2;
    //   future = std::async(std::launch::async, sum of [begin, mid))
    //   sum [mid, end) yourself, then combine with future.get()
    return 0;
}
```

```cpp
long long parallelSum(const std::vector<int>& v) {
    auto mid = v.begin() + static_cast<std::ptrdiff_t>(v.size() / 2);
    auto lower = std::async(std::launch::async, [&v, mid] {
        return std::accumulate(v.begin(), mid, 0LL);
    });
    long long upper = std::accumulate(mid, v.end(), 0LL);  // work while waiting
    return lower.get() + upper;   // blocks; rethrows the task's exception if any
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> v(10000);
    std::iota(v.begin(), v.end(), 1);            // 1..10000
    assert(parallelSum(v) == 50005000LL);
    std::vector<int> odd{3, 1, 4, 1, 5};          // odd length
    assert(parallelSum(odd) == 14);
    std::vector<int> one{42};
    assert(parallelSum(one) == 42);
    assert(parallelSum({}) == 0);                 // empty: async over nothing
    std::puts("PASS");
}
```

**Editorial:** The shape to internalize: launch the async task, *then* do your own share, *then* `get()` — calling `get()` immediately turns "parallel" into "sequential with extra steps". `std::launch::async` is spelled out because the default policy (`async | deferred`) lets the implementation defer the task to run lazily on the calling thread — correct results, zero concurrency, and a classic interview probe. The returned future's `get()` both joins the helper and rethrows any exception the task threw, which is the machinery's quiet superpower over raw `std::thread`. Odd sizes fall out naturally from splitting on iterators; the empty vector works because both halves are empty ranges.
