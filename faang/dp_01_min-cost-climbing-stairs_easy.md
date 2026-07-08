## challenge: Min Cost Climbing Stairs
tags: dynamic-programming, array
track: faang
difficulty: easy

You are given an array `cost` where `cost[i]` is the fee charged when you step on staircase step `i`. From any step you may climb either one or two steps. You may start on step `0` or step `1` for free, and you want to reach the top, which is just past the last step. Return the minimum total fee to reach the top.

Constraints: `2 <= cost.length <= 1000`, `0 <= cost[i] <= 999`.

Example: `cost = [10,15,20]` → `15` (start on step `1`, pay `15`, then climb two steps to the top). Example: `cost = [1,100,1,1,1,100,1,1,100,1]` → `6`.

hint: Think about the cheapest way to arrive at the top, which is reachable from the last step or the one before it.
hint: Let `dp[i]` be the minimum fee needed to stand on step `i`; the top is `dp[n]`.
hint: `dp[i] = min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`, and only the last two values ever matter.

```cpp
// starter
#include <vector>
int minCostClimbingStairs(std::vector<int>& cost);
```

```cpp
int minCostClimbingStairs(std::vector<int>& cost) {
    int n = (int)cost.size();
    int a = 0, b = 0;  // min fee to reach step i-2 and i-1
    for (int i = 2; i <= n; ++i) {
        int cur = std::min(b + cost[i - 1], a + cost[i - 2]);
        a = b;
        b = cur;
    }
    return b;
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
    { vector<int> c{10,15,20}; if (minCostClimbingStairs(c) != 15) { std::puts("case1"); return 1; } }
    { vector<int> c{1,100,1,1,1,100,1,1,100,1}; if (minCostClimbingStairs(c) != 6) { std::puts("case2"); return 1; } }
    { vector<int> c{0,0}; if (minCostClimbingStairs(c) != 0) { std::puts("case3"); return 1; } }
    { vector<int> c{1,2}; if (minCostClimbingStairs(c) != 1) { std::puts("case4"); return 1; } }
    { vector<int> c{10,15}; if (minCostClimbingStairs(c) != 10) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let dp[i] be the minimum fee to stand on step i; the answer is dp[n], the top just past the array. A step is reached from one or two below while paying that step's fee, so dp[i] = min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2]). Two rolling scalars replace the array for O(n) time and O(1) space.
