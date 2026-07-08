## challenge: Contiguous Array
tags: array, hash-table, prefix-sum, arrays-hashing
track: faang
difficulty: medium

Given a binary array `nums` containing only `0`s and `1`s, return the maximum length of a contiguous subarray that holds an equal number of `0`s and `1`s.

Constraints: `1 <= nums.length <= 10^5`, `nums[i]` is `0` or `1`.

Example: `nums = [0,1]` → `2` (the whole array balances). Example: `nums = [0,1,0]` → `2` (either `[0,1]` or `[1,0]`; the full array has two zeros and one one).

hint: Replace every `0` with `-1`; now "equal zeros and ones" means "a subarray that sums to zero".
hint: A zero-sum subarray exists between two indices sharing the same running prefix sum.
hint: Store the earliest index at which each prefix sum first appears; when the sum recurs, the span between the two positions is a balanced subarray — keep the longest.

```cpp
// starter
#include <vector>
int findMaxLength(std::vector<int>& nums);
```

```cpp
int findMaxLength(std::vector<int>& nums) {
    std::unordered_map<int, int> firstIndex;
    firstIndex[0] = -1;  // empty prefix sits before index 0
    int sum = 0, best = 0;
    for (int i = 0; i < (int)nums.size(); ++i) {
        sum += (nums[i] == 1) ? 1 : -1;
        auto it = firstIndex.find(sum);
        if (it != firstIndex.end()) best = std::max(best, i - it->second);
        else firstIndex[sum] = i;
    }
    return best;
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
    { vector<int> n{0,1};                     if (findMaxLength(n) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{0,1,0};                   if (findMaxLength(n) != 2) { std::puts("case2"); return 1; } }
    { vector<int> n{0,0,1,1};                 if (findMaxLength(n) != 4) { std::puts("case3"); return 1; } }
    { vector<int> n{1,1,1,1};                 if (findMaxLength(n) != 0) { std::puts("case4"); return 1; } }
    { vector<int> n{0,1,1,0,1,1,1,0};         if (findMaxLength(n) != 4) { std::puts("case5"); return 1; } }
    { vector<int> n{0};                       if (findMaxLength(n) != 0) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Mapping `0` to `-1` converts the problem into finding the longest subarray with sum zero. Two positions with the same running prefix sum bracket a zero-sum (balanced) subarray, so we record the first index at which each prefix sum occurs and, on every repeat, compare the distance to the stored index. One pass, O(n) time and O(n) space.
