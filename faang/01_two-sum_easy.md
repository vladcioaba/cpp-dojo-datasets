## challenge: Two Sum
tags: array, hash-table, arrays-hashing
track: faang
difficulty: easy

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Exactly one solution exists and you may not use the same element twice. Return the indices in any order.

Constraints: `2 <= nums.length <= 10^4`, `-10^9 <= nums[i], target <= 10^9`.

Example: `nums = [2,7,11,15], target = 9` → `[0,1]` (because `nums[0] + nums[1] == 9`). Example: `nums = [3,2,4], target = 6` → `[1,2]`.

hint: Checking every pair is O(n^2); trade space for time by remembering what you've already seen as you scan.
hint: A hash map from value to index lets you ask "have I already seen `target - nums[i]`?" in O(1).

```cpp
// starter
#include <vector>
std::vector<int> twoSum(std::vector<int>& nums, int target);
```

```cpp
std::vector<int> twoSum(std::vector<int>& nums, int target) {
    std::unordered_map<int, int> seen;
    for (int i = 0; i < (int)nums.size(); ++i) {
        auto it = seen.find(target - nums[i]);
        if (it != seen.end()) return {it->second, i};
        seen[nums[i]] = i;
    }
    return {};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{2,7,11,15}; auto r = twoSum(n, 9); std::sort(r.begin(), r.end());
      if (!(r.size()==2 && r[0]==0 && r[1]==1)) { std::puts("case1"); return 1; } }
    { vector<int> n{3,2,4}; auto r = twoSum(n, 6); std::sort(r.begin(), r.end());
      if (!(r.size()==2 && r[0]==1 && r[1]==2)) { std::puts("case2"); return 1; } }
    { vector<int> n{3,3}; auto r = twoSum(n, 6); std::sort(r.begin(), r.end());
      if (!(r.size()==2 && r[0]==0 && r[1]==1)) { std::puts("case3"); return 1; } }
    { vector<int> n{-1,-2,-3,-4,-5}; auto r = twoSum(n, -8); std::sort(r.begin(), r.end());
      if (!(r.size()==2 && r[0]==2 && r[1]==4)) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** One pass with a hash map. For each element, look up its complement `target - nums[i]`; if it's present you have the answer, otherwise record the current value and index. Average O(1) lookups and inserts make the whole scan O(n) time, O(n) space.
