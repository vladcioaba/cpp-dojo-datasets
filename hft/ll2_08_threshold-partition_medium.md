## challenge: Partition, then the branch is free
tags: branch-prediction, hot-path
track: hft
difficulty: medium

The famous interview riddle — "why is processing a sorted array faster?" — is branch prediction. The production fix is to group data by branch outcome once, so every downstream pass is perfectly predictable. Implement `size_t partitionBelow(int32_t* a, size_t n, int32_t pivot)`: reorder `a` in place so every element `< pivot` comes before every element `>= pivot`, and return the number of elements `< pivot`. Order within each group is unconstrained.

Constraints: `0 <= n <= 10^6`; in place, O(n) time, O(1) extra space, single pass.

Example: `a = [5,1,9,3,7]`, `pivot = 5` → returns `2`; the array becomes some permutation like `[1,3,9,5,7]` where the first 2 elements are `< 5` and the rest are `>= 5`. Elements equal to the pivot belong to the second group.

hint: Keep a write cursor `w` for the "below" region; scan `i` left to right and when `a[i] < pivot`, swap `a[i]` with `a[w]` and advance `w`.
hint: The invariant: at every step `a[0..w)` are all `< pivot` and `a[w..i)` are all `>= pivot` — when the scan ends, `w` is your return value.
hint: Swapping an element with itself (when `i == w`) is harmless — don't special-case it; the branch you'd add costs more than the redundant swap.

```cpp
// starter
#include <cstdint>
#include <cstddef>
size_t partitionBelow(int32_t* a, size_t n, int32_t pivot);
```

```cpp
size_t partitionBelow(int32_t* a, size_t n, int32_t pivot) {
    size_t w = 0;
    for (size_t i = 0; i < n; ++i) {
        if (a[i] < pivot) {
            int32_t t = a[w];
            a[w] = a[i];
            a[i] = t;
            ++w;
        }
    }
    return w;
}
```

```cpp
// harness
#include <cstdio>
#include <cstdint>
#include <cstddef>
#include <algorithm>
//__USER__
static bool verify(int32_t* a, size_t n, int32_t pivot, const int32_t* orig, const char* name) {
    size_t k = partitionBelow(a, n, pivot);
    size_t wantK = 0;
    for (size_t i = 0; i < n; ++i) wantK += (orig[i] < pivot) ? 1u : 0u;
    if (k != wantK) { std::printf("%s: returned %zu want %zu\n", name, k, wantK); return false; }
    for (size_t i = 0; i < k; ++i)
        if (!(a[i] < pivot)) { std::printf("%s: a[%zu]=%d not < pivot\n", name, i, a[i]); return false; }
    for (size_t i = k; i < n; ++i)
        if (a[i] < pivot) { std::printf("%s: a[%zu]=%d should be >= pivot\n", name, i, a[i]); return false; }
    int32_t s1[16], s2[16];
    std::copy(a, a + n, s1);
    std::copy(orig, orig + n, s2);
    std::sort(s1, s1 + n);
    std::sort(s2, s2 + n);
    if (!std::equal(s1, s1 + n, s2)) { std::printf("%s: multiset changed\n", name); return false; }
    return true;
}
int main() {
    { int32_t a[] = {5,1,9,3,7,3,2,8}; int32_t o[8]; std::copy(a,a+8,o); if (!verify(a,8,5,o,"mixed")) return 1; }
    { int32_t a[] = {1,2,3}; int32_t o[3]; std::copy(a,a+3,o); if (!verify(a,3,10,o,"all below")) return 1; }
    { int32_t a[] = {7,8,9}; int32_t o[3]; std::copy(a,a+3,o); if (!verify(a,3,0,o,"none below")) return 1; }
    { int32_t a[] = {4,4,4,4}; int32_t o[4]; std::copy(a,a+4,o); if (!verify(a,4,4,o,"all equal pivot")) return 1; }
    { int32_t a[] = {-3,0,-1,2}; int32_t o[4]; std::copy(a,a+4,o); if (!verify(a,4,0,o,"negatives")) return 1; }
    { int32_t a[1] = {0}; int32_t o[1] = {0}; if (!verify(a,0,5,o,"empty")) return 1; }
    std::puts("PASS");
}
```

**Editorial:** The write-cursor (Lomuto-style) partition maintains a simple invariant — `a[0..w)` holds everything seen so far that is `< pivot`, `a[w..i)` everything that isn't — and each element is examined exactly once with at most one swap, so it's O(n)/O(1) and cache-perfect (two forward-moving pointers the prefetcher loves). The performance story is why this routine exists at all: a downstream loop like `if (x < threshold) hot_path(x);` over *random* data mispredicts ~50% of the time at ~15–20 cycles per miss, which famously makes summing a sorted array several times faster than an unsorted one. Partitioning is the cheaper cousin of sorting — one linear pass buys you two homogeneous ranges where the branch is either always-taken or never-taken, i.e. free. In a trading system this shows up as splitting messages into adds/cancels before processing, or separating in-band from out-of-band prices. For the partition loop itself, `w += (a[i] < pivot)` with an unconditional conditional-swap makes even *this* pass branchless — worth knowing when the partition is the hot loop. `std::partition` does the same job; know what's inside it.
