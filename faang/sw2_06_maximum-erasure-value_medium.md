## challenge: Maximum Erasure Value
tags: array, hash-table, sliding-window
track: faang
difficulty: medium

You are given an array of positive integers `nums`. You may erase exactly one contiguous subarray, but only if all of its elements are distinct. The score of an erasure is the sum of its elements. Return the maximum score achievable, i.e. the largest sum over all subarrays whose elements are all unique.

Constraints: `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^4`.

Example: `nums = [4,2,4,5,6]` → `17` (erase `[2,4,5,6]`). Example: `nums = [5,2,1,2,5,2,1,2,5]` → `8` (erase `[5,2,1]` or `[1,2,5]`). Example: `nums = [10,10,10]` → `10`.

hint: You want the maximum-sum window in which no value repeats — a sliding window with a duplicate check.

hint: Track which values are currently in the window and the running window sum; before admitting a new value, evict from the left until that value is no longer present.

hint: Because values are bounded by `10^4`, a boolean presence array indexed by value gives O(1) membership tests.

```cpp
// starter
#include <vector>
int maximumUniqueSubarray(std::vector<int>& nums);
```

```cpp
int maximumUniqueSubarray(std::vector<int>& nums) {
    std::vector<char> seen(10001, 0);
    int left = 0, sum = 0, best = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        while (seen[nums[right]]) {
            seen[nums[left]] = 0;
            sum -= nums[left];
            ++left;
        }
        seen[nums[right]] = 1;
        sum += nums[right];
        best = std::max(best, sum);
    }
    return best;
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
    { vector<int> n{4,2,4,5,6}; if (maximumUniqueSubarray(n)!=17) { std::puts("case1"); return 1; } }
    { vector<int> n{5,2,1,2,5,2,1,2,5}; if (maximumUniqueSubarray(n)!=8) { std::puts("case2"); return 1; } }
    { vector<int> n{10,10,10}; if (maximumUniqueSubarray(n)!=10) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,3,4,5}; if (maximumUniqueSubarray(n)!=15) { std::puts("case4"); return 1; } }
    { vector<int> n{7}; if (maximumUniqueSubarray(n)!=7) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** This is "longest substring without repeating characters" recast to maximize a sum instead of a length. Maintain a window whose elements are all distinct, along with its running sum and a presence flag per value. When the incoming value already sits in the window, evict elements from the left — clearing their flags and subtracting them from the sum — until the duplicate is gone, then admit the new value and update the best sum. Each element is added and removed at most once, so the scan runs in O(n) time; the presence array over the value range uses O(max value) space.
