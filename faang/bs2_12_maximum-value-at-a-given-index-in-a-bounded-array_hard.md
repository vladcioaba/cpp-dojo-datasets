## challenge: Maximum Value at a Given Index in a Bounded Array
tags: binary-search, greedy
track: faang
difficulty: hard

You are given three integers `n`, `index`, and `maxSum`. Construct an array `nums` of length `n` (0-indexed) satisfying: every `nums[i]` is a positive integer, `|nums[i] - nums[i + 1]| <= 1` for all valid `i`, the total sum is at most `maxSum`, and `nums[index]` is as large as possible. Return that maximum value of `nums[index]`.

Constraints: `1 <= n <= maxSum <= 10^9`, `0 <= index < n`.

Example: `n = 4, index = 2, maxSum = 6` → `2` (e.g. `[1,2,2,1]`). Example: `n = 6, index = 1, maxSum = 10` → `3`.

hint: If a peak value `v` at `index` is achievable, so is any smaller value — the answer is monotone, so binary search `v`.
hint: To minimize the total sum for a given peak `v`, ramp down by 1 each step away from `index` until you hit the floor of `1`, then stay at `1`.
hint: The cost of one side is an arithmetic series that either descends to `1` within the available cells or bottoms out early — compute it in O(1), and watch for overflow with `long long`.

```cpp
// starter
int maxValue(int n, int index, int maxSum);
```

```cpp
int maxValue(int n, int index, int maxSum) {
    // Minimal sum contributed by `count` cells on one side of a peak of value `peak`
    // (values peak-1, peak-2, ... clamped at 1).
    auto sideSum = [](long long peak, long long count) -> long long {
        if (count == 0) return 0;
        if (peak > count) {                     // stays above 1 the whole way
            long long hi = peak - 1, lo = peak - count;
            return (hi + lo) * count / 2;
        } else {                                // descends to 1, then flat 1s
            long long full = (peak - 1) * peak / 2;   // sum of 1..peak-1
            long long ones = count - (peak - 1);
            return full + ones;
        }
    };
    long long left = index, right = n - index - 1;
    long long lo = 1, hi = maxSum;
    while (lo < hi) {
        long long mid = lo + (hi - lo + 1) / 2;       // upper mid, maximize the peak
        long long total = mid + sideSum(mid, left) + sideSum(mid, right);
        if (total <= maxSum) lo = mid;
        else hi = mid - 1;
    }
    return (int)lo;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (maxValue(4, 2, 6)  != 2) { std::puts("case1"); return 1; }
    if (maxValue(6, 1, 10) != 3) { std::puts("case2"); return 1; }
    if (maxValue(1, 0, 1)  != 1) { std::puts("case3"); return 1; }
    if (maxValue(3, 2, 18) != 7) { std::puts("case4"); return 1; }
    if (maxValue(1, 0, 1000000000) != 1000000000) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Achievability is monotone: if peak value `v` fits within `maxSum`, so does any smaller peak, so binary search `v`. For a fixed peak, the minimum-sum array ramps down by 1 on each side until it reaches the floor of `1`, then stays flat — a shape whose per-side cost is either a plain arithmetic series (when the peak is tall enough to stay above `1`) or a triangular sum plus a tail of `1`s. Each side cost is O(1), so the whole search is O(log(maxSum)). Use `long long` throughout: with `n` and `maxSum` up to `10^9`, intermediate products reach ~`10^18`.
