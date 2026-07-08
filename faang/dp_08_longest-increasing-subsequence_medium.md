## challenge: Longest Increasing Subsequence
tags: dynamic-programming, binary-search
track: faang
difficulty: medium

Given an integer array `nums`, return the length of the longest strictly increasing subsequence. A subsequence keeps the original order but may drop elements.

Constraints: `1 <= nums.length <= 2500`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [10,9,2,5,3,7,101,18]` → `4` (one such subsequence is `[2,3,7,101]`). Example: `nums = [7,7,7,7]` → `1`.

hint: Maintain the smallest possible tail value for an increasing subsequence of each length seen so far.
hint: These tail values are always sorted, so the right slot for a new number can be found with binary search.
hint: For each `x`, replace the first tail `>= x` (keeping subsequences short and extendable), or append `x` if it exceeds every tail.

```cpp
// starter
#include <vector>
int lengthOfLIS(std::vector<int>& nums);
```

```cpp
int lengthOfLIS(std::vector<int>& nums) {
    std::vector<int> tails;
    for (int x : nums) {
        auto it = std::lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return (int)tails.size();
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
    { vector<int> n{10,9,2,5,3,7,101,18}; if (lengthOfLIS(n) != 4) { std::puts("case1"); return 1; } }
    { vector<int> n{0,1,0,3,2,3};         if (lengthOfLIS(n) != 4) { std::puts("case2"); return 1; } }
    { vector<int> n{7,7,7,7};             if (lengthOfLIS(n) != 1) { std::puts("case3"); return 1; } }
    { vector<int> n{1};                   if (lengthOfLIS(n) != 1) { std::puts("case4"); return 1; } }
    { vector<int> n{4,10,4,3,8,9};        if (lengthOfLIS(n) != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Keep an array `tails` where `tails[k]` is the smallest value that can end an increasing subsequence of length k+1. Each element either replaces the first tail greater than or equal to it (via binary search) or extends the array, so its length equals the LIS length. This patience-sorting approach runs in O(n log n) time and O(n) space.
