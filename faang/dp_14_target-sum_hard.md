## challenge: Target Sum
tags: dynamic-programming, subset-sum
track: faang
difficulty: hard

Given an array of non-negative integers `nums` and an integer `target`, assign either a `+` or a `-` sign to each number and concatenate them into an expression. Return the number of distinct sign assignments that make the expression evaluate to `target`.

Constraints: `1 <= nums.length <= 20`, `0 <= nums[i] <= 1000`, `0 <= sum(nums) <= 1000`, `-1000 <= target <= 1000`.

Example: `nums = [1,1,1,1,1], target = 3` → `5`. Example: `nums = [1], target = 1` → `1`.

hint: Split the numbers into a positive group `P` and a negative group `N`; then `P - N = target` and `P + N = sum`.
hint: Adding those equations gives `P = (sum + target) / 2`, so the task becomes counting subsets that sum to `P`.
hint: Count subsets with a 0/1 knapsack `dp[s]` scanned downward; note that zeros double the count because `+0` and `-0` both work.

```cpp
// starter
#include <vector>
int findTargetSumWays(std::vector<int>& nums, int target);
```

```cpp
int findTargetSumWays(std::vector<int>& nums, int target) {
    int sum = 0;
    for (int x : nums) sum += x;
    if (target > sum || target < -sum) return 0;
    if ((sum + target) % 2 != 0) return 0;
    int P = (sum + target) / 2;
    std::vector<int> dp(P + 1, 0);
    dp[0] = 1;
    for (int x : nums)
        for (int s = P; s >= x; --s)
            dp[s] += dp[s - x];
    return dp[P];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,1,1,1,1};              if (findTargetSumWays(n, 3) != 5)   { std::puts("case1"); return 1; } }
    { vector<int> n{1};                      if (findTargetSumWays(n, 1) != 1)   { std::puts("case2"); return 1; } }
    { vector<int> n{1};                      if (findTargetSumWays(n, 2) != 0)   { std::puts("case3"); return 1; } }
    { vector<int> n{0,0,0,0,0,0,0,0,1};      if (findTargetSumWays(n, 1) != 256) { std::puts("case4"); return 1; } }
    { vector<int> n{1,0};                    if (findTargetSumWays(n, 1) != 2)   { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Letting P be the sum of the plus-signed numbers, P - N = target and P + N = sum give P = (sum + target) / 2, so the count of valid sign assignments equals the number of subsets summing to P. That subset count is a standard 0/1 knapsack over dp[s] scanned downward, and impossible or non-integer P returns zero. Time is O(n * P), space O(P).
