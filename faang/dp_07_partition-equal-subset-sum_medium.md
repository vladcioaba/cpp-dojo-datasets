## challenge: Partition Equal Subset Sum
tags: dynamic-programming, subset-sum
track: faang
difficulty: medium

Given an array of positive integers `nums`, return `true` if it can be split into two subsets whose sums are equal.

Constraints: `1 <= nums.length <= 200`, `1 <= nums[i] <= 100`.

Example: `nums = [1,5,11,5]` → `true` (`[1,5,5]` and `[11]`). Example: `nums = [1,2,3,5]` → `false`.

hint: Two equal halves are possible only if the total is even, and then each half must sum to `total / 2`.
hint: This reduces to a 0/1 subset-sum: can some subset reach `target = total / 2`?
hint: Use a boolean `dp[s]` = "sum `s` is achievable" and add each number by scanning `s` downward so each item is used at most once.

```cpp
// starter
#include <vector>
bool canPartition(std::vector<int>& nums);
```

```cpp
bool canPartition(std::vector<int>& nums) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % 2 != 0) return false;
    int target = total / 2;
    std::vector<char> dp(target + 1, false);
    dp[0] = true;
    for (int x : nums)
        for (int s = target; s >= x; --s)
            if (dp[s - x]) dp[s] = true;
    return dp[target];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,5,11,5};    if (!canPartition(n)) { std::puts("case1"); return 1; } }
    { vector<int> n{1,2,3,5};     if (canPartition(n))  { std::puts("case2"); return 1; } }
    { vector<int> n{1,1};         if (!canPartition(n)) { std::puts("case3"); return 1; } }
    { vector<int> n{2,2,3,5};     if (canPartition(n))  { std::puts("case4"); return 1; } }
    { vector<int> n{3,3,3,4,5};   if (!canPartition(n)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** If the total is odd no equal split exists; otherwise the problem becomes a 0/1 knapsack asking whether a subset sums to total/2. The boolean dp over reachable sums is updated per item with the sum index scanned downward, which prevents reusing an item within the same pass. Time is O(n * target), space O(target).
