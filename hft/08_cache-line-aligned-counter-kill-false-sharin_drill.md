## challenge: Cache-line aligned counter (kill false sharing)
tags: false-sharing, cache, alignas
track: hft

Two threads incrementing two counters that share a 64-byte cache line ping-pong the line between cores — "false sharing" — and throughput collapses. Fix it by padding each counter onto its own line. Complete `PaddedCounter` so that `sizeof(PaddedCounter) == 64` and `alignof(PaddedCounter) == 64`, with a `long value` member.

```cpp
// starter
struct PaddedCounter {
    // your code: a `long value;` plus padding, aligned to a cache line
};
```

```cpp
struct alignas(64) PaddedCounter {
    long value = 0;
    char pad_[64 - sizeof(long)];
};
```

```cpp
// harness
#include <cstdio>
#include <cstddef>
//__USER__
int main() {
    static_assert(alignof(PaddedCounter) == 64, "must be cache-line aligned");
    static_assert(sizeof(PaddedCounter) == 64, "must fill one cache line");
    PaddedCounter a, b;
    a.value = 5; b.value = 7;
    if (a.value + b.value != 12) { std::puts("value member broken"); return 1; }
    std::puts("PASS");
}
```
