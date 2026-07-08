## challenge: Minimum Difference Between Largest and Smallest Value in Three Moves
tags: two-pointers, array, sorting, greedy
track: faang
difficulty: medium

You are given an integer array `nums`. In one move you may change any single element to any value you like. After performing at most three moves, return the minimum possible difference between the largest and smallest values remaining in the array.

Constraints: `1 <= nums.length <= 10^5`, `-10^9 <= nums[i] <= 10^9`.

Example: `nums = [5,3,2,4]` → `0` (four or fewer elements can all be made equal). Example: `nums = [1,5,0,10,14]` → `1`. Example: `nums = [6,6,0,1,1,4,6]` → `2`.

hint: The three changed elements should always be picked from the current extremes, so sort first.
hint: After sorting, the survivors form a contiguous window of length `n - 3`; you only choose how many extremes to shave off each end.
hint: Try the four splits (drop 0..3 from the front and the rest from the back) and take the smallest max-minus-min over those windows.

```cpp
// starter
#include <vector>
int minDifference(std::vector<int>& nums);
```

```cpp
int minDifference(std::vector<int>& nums) {
    int n = (int)nums.size();
    if (n <= 4) return 0;
    std::sort(nums.begin(), nums.end());
    int res = INT_MAX;
    for (int k = 0; k <= 3; ++k) {
        res = std::min(res, nums[n - 4 + k] - nums[k]);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <climits>
using std::vector;
//__USER__
int main() {
    { vector<int> n{5,3,2,4}; if (minDifference(n) != 0) { std::puts("case1"); return 1; } }
    { vector<int> n{1,5,0,10,14}; if (minDifference(n) != 1) { std::puts("case2"); return 1; } }
    { vector<int> n{6,6,0,1,1,4,6}; if (minDifference(n) != 2) { std::puts("case3"); return 1; } }
    { vector<int> n{1,5,6,14,15}; if (minDifference(n) != 1) { std::puts("case4"); return 1; } }
    { vector<int> n{4,4,4,4}; if (minDifference(n) != 0) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** If four or fewer elements exist, three moves can flatten everything to one value, so the answer is `0`. Otherwise, any element you change is only worth changing if it is an extreme, so sort and consider removing `k` elements from the high end and `3 - k` from the low end for `k = 0..3`. Each choice leaves a contiguous window of `n - 3` sorted values, whose spread is `nums[n-4+k] - nums[k]`. The minimum over the four windows is the answer. O(n log n) time from sorting, O(1) extra space.
