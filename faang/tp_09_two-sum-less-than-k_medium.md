## challenge: Two Sum Less Than K
tags: two-pointers, sorting, array
track: faang
difficulty: medium

Given an array `nums` of integers and an integer `k`, return the maximum sum `nums[i] + nums[j]` such that `i < j` and `nums[i] + nums[j] < k`. If no such pair exists, return `-1`.

Constraints: `1 <= nums.length <= 100`, `1 <= nums[i] <= 1000`, `1 <= k <= 2000`.

Example: `nums = [34,23,1,24,75,33,54,8], k = 60` → `58` (`34 + 24`). Example: `nums = [10,20,30], k = 15` → `-1`.

hint: The pair indices do not matter once you only care about values and their sum, so you are free to reorder the array.
hint: Sort the array and place two pointers at the smallest and largest values.
hint: If the current pair sum is below `k`, record it as a candidate answer and move the low pointer up to try something bigger; otherwise move the high pointer down.

```cpp
// starter
#include <vector>
int twoSumLessThanK(std::vector<int>& nums, int k);
```

```cpp
int twoSumLessThanK(std::vector<int>& nums, int k) {
    std::sort(nums.begin(), nums.end());
    int lo = 0, hi = (int)nums.size() - 1, best = -1;
    while (lo < hi) {
        int s = nums[lo] + nums[hi];
        if (s < k) {
            if (s > best) best = s;
            ++lo;
        } else {
            --hi;
        }
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
    { vector<int> n{34,23,1,24,75,33,54,8}; if (twoSumLessThanK(n, 60) != 58) { std::puts("case1"); return 1; } }
    { vector<int> n{10,20,30};              if (twoSumLessThanK(n, 15) != -1) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3,4};               if (twoSumLessThanK(n, 7) != 6) { std::puts("case3"); return 1; } }
    { vector<int> n{5,5};                   if (twoSumLessThanK(n, 11) != 10) { std::puts("case4"); return 1; } }
    { vector<int> n{5,5};                   if (twoSumLessThanK(n, 10) != -1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort the values so ordering is monotone, then converge two pointers from both ends. When the current sum is already below `k`, it is a valid candidate and, because the array is sorted, the only way to increase it is to advance the low pointer; when the sum is `k` or larger, the high pointer must retreat to shrink it. Tracking the best valid sum along the way gives the answer in O(n log n) time (dominated by the sort) and O(1) extra space.
