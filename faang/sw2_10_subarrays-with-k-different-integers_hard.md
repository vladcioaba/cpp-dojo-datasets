## challenge: Subarrays with K Different Integers
tags: array, hash-table, sliding-window, counting
track: faang
difficulty: hard

Given an integer array `nums` and an integer `k`, return the number of *good* contiguous subarrays, where a subarray is good if it contains exactly `k` distinct integers.

Constraints: `1 <= nums.length <= 2 * 10^4`, `1 <= nums[i] <= nums.length`, `1 <= k <= nums.length`.

Example: `nums = [1,2,1,2,3], k = 2` → `7` (the good subarrays are `[1,2]`, `[1,2,1]`, `[1,2,1,2]`, `[2,1]`, `[2,1,2]`, `[1,2]`, and `[2,3]`). Example: `nums = [1,2,1,3,4], k = 3` → `3` (the subarrays `[1,2,1,3]`, `[2,1,3]`, `[1,3,4]`).

hint: "Exactly `k` distinct" resists a single window because shrinking to keep the count at exactly `k` is not well defined.

hint: Reframe it as a difference of two monotone quantities: `exactly(k) = atMost(k) - atMost(k - 1)`.

hint: For `atMost(m)`, slide a window keeping its distinct count within `m`; each right edge contributes `right - left + 1` subarrays, all with at most `m` distinct values.

```cpp
// starter
#include <vector>
int subarraysWithKDistinct(std::vector<int>& nums, int k);
```

```cpp
static long long atMostK(std::vector<int>& nums, int k) {
    std::unordered_map<int, int> cnt;
    int left = 0, distinct = 0;
    long long count = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        if (cnt[nums[right]]++ == 0) ++distinct;
        while (distinct > k) {
            if (--cnt[nums[left]] == 0) --distinct;
            ++left;
        }
        count += right - left + 1;
    }
    return count;
}
int subarraysWithKDistinct(std::vector<int>& nums, int k) {
    return (int)(atMostK(nums, k) - atMostK(nums, k - 1));
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,2,1,2,3}; if (subarraysWithKDistinct(n,2)!=7) { std::puts("case1"); return 1; } }
    { vector<int> n{1,2,1,3,4}; if (subarraysWithKDistinct(n,3)!=3) { std::puts("case2"); return 1; } }
    { vector<int> n{1,1,1,1}; if (subarraysWithKDistinct(n,1)!=10) { std::puts("case3"); return 1; } }
    { vector<int> n{2,1,1,1,2}; if (subarraysWithKDistinct(n,2)!=7) { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,3,4,5}; if (subarraysWithKDistinct(n,5)!=1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Counting subarrays with *exactly* `k` distinct integers directly is hard because a window cannot be shrunk to hold precisely `k` distinct values in a well-defined way. The standard resolution is `exactly(k) = atMost(k) - atMost(k - 1)`: both terms are computable with a monotone sliding window. In `atMost(m)`, extend the right edge and, whenever the distinct count exceeds `m`, retract the left edge until it fits; every valid right edge then contributes `right - left + 1` subarrays. Two O(n) passes with a hash map yield O(n) time overall.
