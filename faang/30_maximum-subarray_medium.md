## challenge: Maximum Subarray
tags: array, dynamic-programming, greedy
track: faang
difficulty: medium

Given an integer array `nums`, find the contiguous non-empty subarray with the largest sum and return that sum.

Constraints: `1 <= nums.length <= 10^5`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [-2,1,-3,4,-1,2,1,-5,4]` → `6` (the subarray `[4,-1,2,1]`). Example: `nums = [1]` → `1`. Example: `nums = [-3,-1,-2]` → `-1` (best is a single element when all are negative).

hint: A prefix that has become negative can only hurt whatever follows it.
hint: This is Kadane's algorithm — track the best subarray sum ending at the current index.
hint: At each element choose to extend the running sum or restart from that element: `cur = max(nums[i], cur + nums[i])`, and keep the maximum `cur` seen.

```cpp
// starter
#include <vector>
int maxSubArray(std::vector<int>& nums);
```

```cpp
int maxSubArray(std::vector<int>& nums) {
    int best = nums[0];
    int cur = nums[0];
    for (int i = 1; i < (int)nums.size(); ++i) {
        cur = std::max(nums[i], cur + nums[i]);
        best = std::max(best, cur);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{-2,1,-3,4,-1,2,1,-5,4}; if (maxSubArray(n) != 6)  { std::puts("case1"); return 1; } }
    { vector<int> n{1};                     if (maxSubArray(n) != 1)  { std::puts("case2"); return 1; } }
    { vector<int> n{-3,-1,-2};              if (maxSubArray(n) != -1) { std::puts("case3"); return 1; } }
    { vector<int> n{5,4,-1,7,8};            if (maxSubArray(n) != 23) { std::puts("case4"); return 1; } }
    { vector<int> n{-5};                    if (maxSubArray(n) != -5) { std::puts("case5"); return 1; } }
    { vector<int> n{-2,-1};                 if (maxSubArray(n) != -1) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Kadane's algorithm keeps `cur`, the maximum sum of a subarray ending at the current index: extend the previous run or start fresh at `nums[i]`, whichever is larger. The global answer is the largest `cur` observed. Seeding both `best` and `cur` with `nums[0]` handles the all-negative case correctly, giving O(n) time and O(1) space.
