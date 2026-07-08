## challenge: Branchless count of elements >= threshold
tags: branchless, low-level-optimization, hot-loop
track: hft
difficulty: hard

Counting how many array elements meet a threshold is a classic hot loop (how many quotes cross a price?). Written as `if (data[i] >= t) ++count;` the per-element `if` is data-dependent, and on random data it mispredicts ~50% of the time — ~15-20 cycles per miss, dwarfing the work. Implement `int count_ge(const int* data, int n, int t)` with no data-dependent branch in the loop body, so it stays predictable and vectorizes.

Constraints: `n >= 0`; `data` has `n` elements; each element and `t` are any 32-bit `int` including `INT_MIN`/`INT_MAX`. Compare directly (no `data[i] - t`, which could overflow). The loop's own `i < n` bound is fine — it is perfectly predicted; only the per-element decision must be branchless.

Example: `count_ge({1,5,3,8,2}, 5, 3)` → `3` (5, 3, 8). Example: with `t = INT_MIN` → `5` (all). Example: with `t = INT_MAX` and one `INT_MAX` present → count of those equal to `INT_MAX`.

hint: A comparison already yields `0` or `1` — you do not need an `if` to turn "meets threshold" into a count contribution.
hint: Accumulate the boolean directly: `count += (data[i] >= t);` — no jump, so nothing per element to mispredict.
hint: Because each iteration is a compare producing `0`/`1` plus an add, the compiler can `setcc` it (and often SIMD-widen the reduction); the only branch left is the loop bound.

```cpp
// starter
int count_ge(const int* data, int n, int t);
```

```cpp
int count_ge(const int* data, int n, int t) {
    int count = 0;
    for (int i = 0; i < n; ++i) {
        count += (data[i] >= t);       // 0 or 1, no data-dependent branch
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
int main() {
    int a[] = {1, 5, 3, 8, 2};
    if (count_ge(a, 5, 3) != 3) { std::puts("case1"); return 1; }
    if (count_ge(a, 5, INT_MIN) != 5) { std::puts("case2"); return 1; }
    if (count_ge(a, 5, 9) != 0) { std::puts("case3"); return 1; }
    if (count_ge(a, 0, 0) != 0) { std::puts("case4 empty"); return 1; }
    int b[] = {INT_MIN, INT_MAX, 0, INT_MAX, -1};
    if (count_ge(b, 5, INT_MAX) != 2) { std::puts("case5"); return 1; }
    if (count_ge(b, 5, INT_MIN) != 5) { std::puts("case6"); return 1; }
    if (count_ge(b, 5, 0) != 3) { std::puts("case7"); return 1; }   // INT_MAX,0,INT_MAX
    std::puts("PASS");
}
```

**Editorial:** The comparison `data[i] >= t` is already a `0`/`1` value, so `count += (data[i] >= t)` folds the decision into arithmetic — there is no per-element conditional jump to mispredict. On random data the naive `if (...) ++count` mispredicts about half the time at ~15-20 cycles per miss, which can cost more than the loop's real work; the branchless form keeps the pipeline full and lets the compiler emit a `setcc`/`cmov` and even SIMD-widen the reduction (comparing and summing several lanes at once). Comparing directly rather than testing `data[i] - t >= 0` avoids signed-overflow UB for far-apart values. The remaining `i < n` branch is loop control and is predicted essentially perfectly. O(n) time, O(1) space.
