## challenge: fix: range totals come up short
tags: code-review, debugging, bounds
track: core
difficulty: easy

This code review found a bug: totals over an inclusive index range are consistently short — the last element of the range is never counted. Find and fix it — keep the function signature.

hint: Look at the loop condition, not the accumulation.
hint: This is an off-by-one: an inclusive bound treated as exclusive.
hint: The contract says v[lo..hi] inclusive, but the loop stops when i == hi, so v[hi] is never added.

```cpp
// starter
// Returns the sum of v[lo..hi], both endpoints inclusive.
// Precondition: lo <= hi < v.size().
int sumRange(const std::vector<int>& v, std::size_t lo, std::size_t hi) {
    int total = 0;
    for (std::size_t i = lo; i < hi; ++i) {
        total += v[i];
    }
    return total;
}
```

```cpp
// Returns the sum of v[lo..hi], both endpoints inclusive.
// Precondition: lo <= hi < v.size().
int sumRange(const std::vector<int>& v, std::size_t lo, std::size_t hi) {
    int total = 0;
    for (std::size_t i = lo; i <= hi; ++i) {
        total += v[i];
    }
    return total;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> v{10, 20, 30, 40};
    assert(sumRange(v, 1, 3) == 90);   // 20 + 30 + 40
    assert(sumRange(v, 0, 0) == 10);   // single-element range
    assert(sumRange(v, 0, 3) == 100);  // whole vector
    std::puts("PASS");
}
```

**Editorial:** The documented contract is inclusive on both ends, but the loop condition `i < hi` stops one element early, so `v[hi]` is silently dropped — a single-element range even sums to zero. The fix is `i <= hi` (safe here because the precondition guarantees `hi < v.size()`). A reviewer spots this by matching the comment's contract against the loop bound: whenever a range is described as inclusive, `<` in the terminating condition is the first suspect.
