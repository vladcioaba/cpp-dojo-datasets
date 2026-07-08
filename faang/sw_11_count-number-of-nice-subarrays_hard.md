## challenge: Count Number of Nice Subarrays
tags: array, sliding-window, prefix-sum
track: faang
difficulty: hard

An array is "nice" if it contains exactly `k` odd numbers. Given an integer array `nums` and an integer `k`, return the number of contiguous subarrays that are nice — that is, contain exactly `k` odd numbers.

Constraints: `1 <= nums.length <= 5 * 10^4`, `1 <= nums[i] <= 10^5`, `1 <= k <= nums.length`.

Example: `nums = [1,1,2,1,1], k = 3` → `2` (`[1,1,2,1]` and `[1,2,1,1]`). Example: `nums = [2,4,6], k = 1` → `0`. Example: `nums = [2,2,2,1,2,2,1,2,2,2], k = 2` → `16`.

hint: "Exactly `k`" is awkward to count directly, but "at most `k`" is easy with a window that shrinks once it holds more than `k` odds.
hint: The number of subarrays with exactly `k` odds equals atMost(`k`) minus atMost(`k - 1`).
hint: In `atMost(k)`, for each right end add `right - left + 1` valid subarrays ending there, shrinking left whenever the odd count exceeds the bound.

```cpp
// starter
#include <vector>
int numberOfSubarrays(std::vector<int>& nums, int k);
```

```cpp
int numberOfSubarrays(std::vector<int>& nums, int k) {
    auto atMost = [&](int m) {
        if (m < 0) return 0;
        int left = 0, odds = 0, res = 0;
        for (int right = 0; right < (int)nums.size(); ++right) {
            odds += nums[right] & 1;
            while (odds > m) odds -= nums[left++] & 1;
            res += right - left + 1;
        }
        return res;
    };
    return atMost(k) - atMost(k - 1);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,1,2,1,1}; if (numberOfSubarrays(n,3)!=2) { std::puts("case1"); return 1; } }
    { vector<int> n{2,4,6}; if (numberOfSubarrays(n,1)!=0) { std::puts("case2"); return 1; } }
    { vector<int> n{2,2,2,1,2,2,1,2,2,2}; if (numberOfSubarrays(n,2)!=16) { std::puts("case3"); return 1; } }
    { vector<int> n{1,1,1,1,1}; if (numberOfSubarrays(n,1)!=5) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Counting subarrays with *exactly* `k` odd numbers is easier as a difference of two "at most" counts: exactly(k) = atMost(k) - atMost(k-1). For `atMost(m)`, run a sliding window that keeps the number of odd values at most `m`; every time the right end advances, add `right - left + 1`, which is the number of subarrays ending at `right` that stay within the bound. Two linear passes give O(n) time and O(1) space. (Treating odd numbers as `1` and evens as `0`, this is equivalently "subarrays with sum exactly `k`".)
