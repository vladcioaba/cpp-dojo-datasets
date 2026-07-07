## challenge: House Robber

tags: array, dynamic-programming
track: faang
difficulty: medium

You are a robber planning to loot houses along a street, where `nums[i]` is the money in house `i`. You cannot rob two adjacent houses on the same night without triggering an alarm. Return the maximum total amount you can rob.

Constraints: `0 <= nums.length <= 100`, `0 <= nums[i] <= 400`.

Example: `nums = [1,2,3,1]` → `4` (rob houses `0` and `2`). Example: `nums = [2,7,9,3,1]` → `12` (rob houses `0`, `2`, and `4`).

hint: At each house you either skip it (keep the best so far) or rob it (add its value to the best from two houses back).
hint: This is a linear DP; the state is only the best totals ending at the previous two positions.
hint: Track `cur` and `prev`; the new best is `max(cur, prev + nums[i])`, then shift the window forward.

```cpp
// starter
#include <vector>
int rob(std::vector<int>& nums);
```

```cpp
int rob(std::vector<int>& nums) {
    int prev = 0, cur = 0;  // best totals two-back and one-back
    for (int x : nums) {
        int next = std::max(cur, prev + x);
        prev = cur;
        cur = next;
    }
    return cur;
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
    { vector<int> n{1,2,3,1};   if (rob(n) != 4)  { std::puts("case1"); return 1; } }
    { vector<int> n{2,7,9,3,1}; if (rob(n) != 12) { std::puts("case2"); return 1; } }
    { vector<int> n{5};         if (rob(n) != 5)  { std::puts("case3"); return 1; } }
    { vector<int> n{};          if (rob(n) != 0)  { std::puts("case4"); return 1; } }
    { vector<int> n{2,1,1,2};   if (rob(n) != 4)  { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let the best loot ending at each house be the larger of skipping it (the previous best) or robbing it (its value plus the best from two houses earlier). Only the last two running maxima matter, so two scalars replace the full DP array. One left-to-right pass gives O(n) time and O(1) space.
