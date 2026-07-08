## challenge: Binary Subarrays With Sum
tags: array, hash-table, sliding-window, prefix-sum
track: faang
difficulty: medium

Given a binary array `nums` (containing only `0`s and `1`s) and an integer `goal`, return the number of non-empty contiguous subarrays whose elements sum to exactly `goal`.

Constraints: `1 <= nums.length <= 3 * 10^4`, `nums[i]` is `0` or `1`, `0 <= goal <= nums.length`.

Example: `nums = [1,0,1,0,1], goal = 2` → `4` (the qualifying subarrays span indices `0..2`, `0..3`, `1..4`, and `2..4`). Example: `nums = [0,0,0,0,0], goal = 0` → `15` (every one of the 15 subarrays sums to zero).

hint: Counting subarrays with sum exactly `goal` is easier as a difference: (subarrays with sum at most `goal`) minus (subarrays with sum at most `goal - 1`).

hint: For "at most `x`", slide a window and keep the sum `<= x` by advancing the left edge; each new right edge then contributes `right - left + 1` valid subarrays.

hint: The presence of `0`s (which can pad a window without changing its sum) is exactly why the plain "exactly" window is awkward and the at-most difference is clean.

```cpp
// starter
#include <vector>
int numSubarraysWithSum(std::vector<int>& nums, int goal);
```

```cpp
static int atMostSum(std::vector<int>& nums, int goal) {
    if (goal < 0) return 0;
    int left = 0, sum = 0, count = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        sum += nums[right];
        while (sum > goal) sum -= nums[left++];
        count += right - left + 1;
    }
    return count;
}
int numSubarraysWithSum(std::vector<int>& nums, int goal) {
    return atMostSum(nums, goal) - atMostSum(nums, goal - 1);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,0,1,0,1}; if (numSubarraysWithSum(n,2)!=4) { std::puts("case1"); return 1; } }
    { vector<int> n{0,0,0,0,0}; if (numSubarraysWithSum(n,0)!=15) { std::puts("case2"); return 1; } }
    { vector<int> n{1,1,1,1,1}; if (numSubarraysWithSum(n,3)!=3) { std::puts("case3"); return 1; } }
    { vector<int> n{0,1,0,1,0}; if (numSubarraysWithSum(n,1)!=8) { std::puts("case4"); return 1; } }
    { vector<int> n{1,0,0,0,1}; if (numSubarraysWithSum(n,0)!=6) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A window sum with only non-negative entries is monotonic, but the `0`s let many different windows share the same sum, so a direct "exactly `goal`" window double-counts awkwardly. The clean trick is `exactly(goal) = atMost(goal) - atMost(goal - 1)`. For `atMost(x)`, slide a window keeping its sum at most `x`; each time the right edge moves, every subarray ending there and starting at or after `left` is valid, adding `right - left + 1` to the count. Two linear passes give O(n) time and O(1) space.
