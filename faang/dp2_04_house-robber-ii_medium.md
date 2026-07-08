## challenge: House Robber II
tags: dynamic-programming, array
track: faang
difficulty: medium

Houses are arranged in a circle, so the first house is adjacent to the last. Each house holds `nums[i]` dollars, and robbing two adjacent houses on the same night triggers the alarm. Return the maximum amount you can rob without alerting the police.

Constraints: `1 <= nums.length <= 100`, `0 <= nums[i] <= 1000`.

Example: `nums = [2,3,2]` → `3` (you cannot take both house `0` and house `2` because they are adjacent). Example: `nums = [1,2,3,1]` → `4` (rob houses `0` and `2`).

hint: The circle means house `0` and the last house can never both be robbed.
hint: Run the linear House Robber twice — once excluding the last house, once excluding the first — and take the better result.

```cpp
// starter
#include <vector>
int rob(std::vector<int>& nums);
```

```cpp
static int robLine(const std::vector<int>& nums, int lo, int hi) {
    int prev = 0, cur = 0;
    for (int i = lo; i <= hi; ++i) {
        int next = std::max(cur, prev + nums[i]);
        prev = cur;
        cur = next;
    }
    return cur;
}
int rob(std::vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;
    if (n == 1) return nums[0];
    return std::max(robLine(nums, 0, n - 2), robLine(nums, 1, n - 1));
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
    { vector<int> n{2,3,2};            if (rob(n) != 3)   { std::puts("case1"); return 1; } }
    { vector<int> n{1,2,3,1};          if (rob(n) != 4)   { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3};            if (rob(n) != 3)   { std::puts("case3"); return 1; } }
    { vector<int> n{1};                if (rob(n) != 1)   { std::puts("case4"); return 1; } }
    { vector<int> n{200,3,140,20,10};  if (rob(n) != 340) { std::puts("case5"); return 1; } }
    { vector<int> n{2,7,9,3,1};        if (rob(n) != 11)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The circular adjacency forbids taking the first and last house together, so every valid plan lives entirely in `nums[0..n-2]` or entirely in `nums[1..n-1]`. Each of those windows is an ordinary linear House Robber problem — track the best totals for skipping versus robbing each house with two scalars — and the answer is the larger of the two runs. The `n == 1` edge case is handled directly. O(n) time, O(1) space.
