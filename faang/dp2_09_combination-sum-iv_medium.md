## challenge: Combination Sum IV
tags: dynamic-programming, array
track: faang
difficulty: medium

Given an array `nums` of distinct positive integers and a target integer `target`, return the number of ordered combinations (sequences) whose elements sum to `target`. Different orderings of the same multiset count as different combinations, and each number may be reused any number of times.

Constraints: `1 <= nums.length <= 200`, `1 <= nums[i] <= 1000`, all `nums[i]` distinct, `1 <= target <= 1000`. The answer is guaranteed to fit in a 32-bit signed integer.

Example: `nums = [1,2,3], target = 4` → `7` (the sequences `1+1+1+1`, `1+1+2`, `1+2+1`, `2+1+1`, `2+2`, `1+3`, `3+1`). Example: `nums = [9], target = 3` → `0`.

hint: Since order matters, think about which number comes last in a sequence summing to `t`.
hint: `dp[t] = sum over x in nums of dp[t-x]`, with `dp[0] = 1` as the empty sequence.

```cpp
// starter
#include <vector>
int combinationSum4(std::vector<int>& nums, int target);
```

```cpp
int combinationSum4(std::vector<int>& nums, int target) {
    std::vector<unsigned long long> dp(target + 1, 0);
    dp[0] = 1;
    for (int t = 1; t <= target; ++t)
        for (int x : nums)
            if (x <= t) dp[t] += dp[t - x];
    return (int)dp[target];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,2,3}; if (combinationSum4(n, 4)  != 7)  { std::puts("case1"); return 1; } }
    { vector<int> n{9};     if (combinationSum4(n, 3)  != 0)  { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3}; if (combinationSum4(n, 1)  != 1)  { std::puts("case3"); return 1; } }
    { vector<int> n{2,3};   if (combinationSum4(n, 7)  != 3)  { std::puts("case4"); return 1; } }
    { vector<int> n{1,2};   if (combinationSum4(n, 5)  != 8)  { std::puts("case5"); return 1; } }
    { vector<int> n{4,2,1}; if (combinationSum4(n, 32) != 39882198) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because ordering matters, classify each sequence by its final element `x`: removing it leaves an ordered sequence summing to `t - x`. Summing over every choice of last element gives `dp[t] = sum of dp[t-x]` for each `x <= t`, seeded by `dp[0] = 1` (the one empty sequence). Iterating targets in the outer loop and numbers in the inner loop counts permutations rather than sets. Intermediate counts use unsigned 64-bit to stay safe. O(target · |nums|) time, O(target) space.
