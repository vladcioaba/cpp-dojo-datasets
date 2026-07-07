## challenge: 3Sum
tags: two-pointers, array, sorting
track: faang
difficulty: medium

Given an integer array `nums`, return all unique triplets `[nums[i], nums[j], nums[k]]` with distinct indices such that they sum to zero. The solution set must not contain duplicate triplets. Return in any order.

Constraints: `3 <= nums.length <= 3000`, `-10^5 <= nums[i] <= 10^5`.

Example: `nums = [-1,0,1,2,-1,-4]` → `[[-1,-1,2],[-1,0,1]]`. Example: `nums = [0,0,0]` → `[[0,0,0]]`. Example: `nums = [1,2,3]` → `[]`.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> threeSum(std::vector<int>& nums);
```

```cpp
std::vector<std::vector<int>> threeSum(std::vector<int>& nums) {
    std::sort(nums.begin(), nums.end());
    int n = (int)nums.size();
    std::vector<std::vector<int>> res;
    for (int i = 0; i < n - 2; ++i) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        int l = i + 1, r = n - 1;
        while (l < r) {
            int s = nums[i] + nums[l] + nums[r];
            if (s < 0) ++l;
            else if (s > 0) --r;
            else {
                res.push_back({nums[i], nums[l], nums[r]});
                ++l; --r;
                while (l < r && nums[l] == nums[l - 1]) ++l;
                while (l < r && nums[r] == nums[r + 1]) --r;
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
    { vector<int> n{-1,0,1,2,-1,-4};
      if (canon(threeSum(n)) != canon({{-1,-1,2},{-1,0,1}})) { std::puts("case1"); return 1; } }
    { vector<int> n{0,0,0};
      if (canon(threeSum(n)) != canon({{0,0,0}})) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3};
      if (!threeSum(n).empty()) { std::puts("case3"); return 1; } }
    { vector<int> n{-2,0,1,1,2};
      if (canon(threeSum(n)) != canon({{-2,0,2},{-2,1,1}})) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```
