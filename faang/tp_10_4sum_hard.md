## challenge: 4Sum
tags: two-pointers, array, sorting
track: faang
difficulty: hard

Given an integer array `nums` and an integer `target`, return all unique quadruplets `[nums[a], nums[b], nums[c], nums[d]]` such that `a`, `b`, `c`, `d` are distinct indices and `nums[a] + nums[b] + nums[c] + nums[d] == target`. The solution set must not contain duplicate quadruplets. Return them in any order.

Constraints: `1 <= nums.length <= 200`, `-10^9 <= nums[i] <= 10^9`, `-10^9 <= target <= 10^9`. Note that a sum of four values can overflow 32-bit arithmetic, so accumulate in 64-bit.

Example: `nums = [1,0,-1,0,-2,2], target = 0` → `[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]`. Example: `nums = [2,2,2,2,2], target = 8` → `[[2,2,2,2]]`.

hint: 4Sum generalizes 3Sum: fix two of the numbers, and the rest reduces to a two-pointer search for the remaining pair.
hint: Sort the array, loop over the first two indices, and for each fixed pair run converging pointers over the suffix.
hint: Skip equal adjacent values at every level (both fixed indices and both pointers) to avoid duplicates, and use a 64-bit accumulator so the four-way sum never overflows.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> fourSum(std::vector<int>& nums, int target);
```

```cpp
std::vector<std::vector<int>> fourSum(std::vector<int>& nums, int target) {
    std::sort(nums.begin(), nums.end());
    int n = (int)nums.size();
    std::vector<std::vector<int>> res;
    for (int i = 0; i < n - 3; ++i) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        for (int j = i + 1; j < n - 2; ++j) {
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;
            long long need = (long long)target - nums[i] - nums[j];
            int l = j + 1, r = n - 1;
            while (l < r) {
                long long s = (long long)nums[l] + nums[r];
                if (s < need) ++l;
                else if (s > need) --r;
                else {
                    res.push_back({nums[i], nums[j], nums[l], nums[r]});
                    ++l; --r;
                    while (l < r && nums[l] == nums[l - 1]) ++l;
                    while (l < r && nums[r] == nums[r + 1]) --r;
                }
            }
        }
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
static vector<vector<int>> canon(vector<vector<int>> g) {
    for (auto& row : g) std::sort(row.begin(), row.end());
    std::sort(g.begin(), g.end());
    return g;
}
//__USER__
int main() {
    { vector<int> n{1,0,-1,0,-2,2};
      if (canon(fourSum(n, 0)) != canon({{-2,-1,1,2},{-2,0,0,2},{-1,0,0,1}})) { std::puts("case1"); return 1; } }
    { vector<int> n{2,2,2,2,2};
      if (canon(fourSum(n, 8)) != canon({{2,2,2,2}})) { std::puts("case2"); return 1; } }
    { vector<int> n{0,0,0,0};
      if (canon(fourSum(n, 1)) != canon({})) { std::puts("case3"); return 1; } }
    { vector<int> n{-2,-1,-1,1,1,2,2};
      if (canon(fourSum(n, 0)) != canon({{-2,-1,1,2},{-1,-1,1,1}})) { std::puts("case4"); return 1; } }
    { vector<int> n{1000000000,1000000000,1000000000,1000000000};
      if (canon(fourSum(n, -294967296)) != canon({})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** 4Sum is a nested reduction of 3Sum. Sort the array, then fix the two outer indices with a double loop; for each fixed pair the problem becomes finding a pair in the sorted suffix that sums to `target - nums[i] - nums[j]`, which the classic two-pointer converge solves in linear time. Duplicate quadruplets are avoided by skipping equal adjacent values at all four positions. The critical correctness detail is 64-bit accumulation: four values near 10^9 exceed the 32-bit range, so the sums are computed as `long long`. Two outer loops times a linear inner scan give O(n^3) time and O(1) extra space beyond the output.
