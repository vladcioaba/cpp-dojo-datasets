## challenge: Find All Duplicates in an Array
tags: array, hash-table, arrays-hashing
track: faang
difficulty: hard

Given an integer array `nums` of length `n` where all integers are in the range `[1, n]` and each integer appears once or twice, return an array of all the integers that appear twice. You must solve it in O(n) time and use only constant extra space (not counting the output).

Constraints: `n == nums.length`, `1 <= n <= 10^5`, `1 <= nums[i] <= n`, each element appears once or twice.

Example: `nums = [4,3,2,7,8,2,3,1]` → `[2,3]`. Example: `nums = [1,1,2]` → `[1]`. Example: `nums = [1]` → `[]`.

hint: The values themselves are valid indices (after subtracting 1), so the array can encode "have I seen this value" in place.
hint: For each value `v`, visit index `abs(v) - 1` and flip the sign of the entry stored there to mark the value as seen.
hint: If you arrive at an index that is already negative, the corresponding value has been seen before, so it is a duplicate.

```cpp
// starter
#include <vector>
std::vector<int> findDuplicates(std::vector<int>& nums);
```

```cpp
std::vector<int> findDuplicates(std::vector<int>& nums) {
    std::vector<int> res;
    for (int i = 0; i < (int)nums.size(); ++i) {
        int idx = std::abs(nums[i]) - 1;
        if (nums[idx] < 0) res.push_back(idx + 1);
        else nums[idx] = -nums[idx];
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <cstdlib>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{4,3,2,7,8,2,3,1}; auto r = findDuplicates(n); std::sort(r.begin(), r.end());
      if (!(r.size()==2 && r[0]==2 && r[1]==3)) { std::puts("case1"); return 1; } }
    { vector<int> n{1,1,2}; auto r = findDuplicates(n); std::sort(r.begin(), r.end());
      if (!(r.size()==1 && r[0]==1)) { std::puts("case2"); return 1; } }
    { vector<int> n{1}; auto r = findDuplicates(n);
      if (!r.empty()) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,3,3,2}; auto r = findDuplicates(n); std::sort(r.begin(), r.end());
      if (!(r.size()==2 && r[0]==2 && r[1]==3)) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because every value lies in `[1, n]`, each value doubles as an index into the array itself. Use the sign of `nums[abs(v)-1]` as a one-bit "seen" flag: the first visit flips it negative, and a second visit finds it already negative, revealing the duplicate. Taking absolute values keeps the original magnitudes readable despite the sign flips. This encodes a hash set inside the input for O(n) time and O(1) extra space.
