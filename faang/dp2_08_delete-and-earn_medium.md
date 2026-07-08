## challenge: Delete and Earn
tags: dynamic-programming, array, hash-table
track: faang
difficulty: medium

Given an integer array `nums`, you repeatedly pick any element `nums[i]`, earn `nums[i]` points, and then must delete every element equal to `nums[i] - 1` and every element equal to `nums[i] + 1`. Return the maximum number of points you can earn by performing this operation any number of times.

Constraints: `1 <= nums.length <= 2 * 10^4`, `1 <= nums[i] <= 10^4`.

Example: `nums = [3,4,2]` → `6` (take `4` to earn 4 and delete the `3`s, then take `2` to earn 2). Example: `nums = [2,2,3,3,3,4]` → `9` (take all three `3`s for 9, which deletes every `2` and `4`).

hint: Taking any copy of a value `v` takes them all and forbids values `v-1` and `v+1` — that is House Robber over the value axis.
hint: Build `total[v]` = sum of all elements equal to `v`, then choose a subset of values with no two consecutive.

```cpp
// starter
#include <vector>
int deleteAndEarn(std::vector<int>& nums);
```

```cpp
int deleteAndEarn(std::vector<int>& nums) {
    if (nums.empty()) return 0;
    int mx = *std::max_element(nums.begin(), nums.end());
    std::vector<long long> total(mx + 1, 0);
    for (int x : nums) total[x] += x;
    long long prev = 0, cur = 0;  // best up to v-2 and v-1
    for (int v = 0; v <= mx; ++v) {
        long long take = prev + total[v];
        long long skip = cur;
        prev = cur;
        cur = std::max(take, skip);
    }
    return (int)cur;
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
    { vector<int> n{3,4,2};             if (deleteAndEarn(n) != 6)  { std::puts("case1"); return 1; } }
    { vector<int> n{2,2,3,3,3,4};       if (deleteAndEarn(n) != 9)  { std::puts("case2"); return 1; } }
    { vector<int> n{1};                 if (deleteAndEarn(n) != 1)  { std::puts("case3"); return 1; } }
    { vector<int> n{3,1};               if (deleteAndEarn(n) != 4)  { std::puts("case4"); return 1; } }
    { vector<int> n{1,1,1,2,4,5,5,5,6}; if (deleteAndEarn(n) != 18) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Deleting neighbors means the decision to earn value `v` is all-or-nothing across its copies, and it locks out `v-1` and `v+1`. Collapse the array into `total[v]`, the summed payoff of value `v`, laid out on the number line. The task becomes: pick values with no two adjacent to maximize the sum of `total` — exactly House Robber. Sweep `v` from 0 to the maximum, keeping the best totals two-back and one-back, and choose `max(skip, take)` at each value. O(n + max) time.
