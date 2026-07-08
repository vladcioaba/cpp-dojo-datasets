## challenge: First Missing Positive
tags: array, hash-table, arrays-hashing
track: faang
difficulty: hard

Given an unsorted integer array `nums`, return the smallest positive integer (starting from `1`) that does not appear in the array. You must achieve O(n) time and use O(1) auxiliary space beyond the input array.

Constraints: `1 <= nums.length <= 10^5`, `-2^31 <= nums[i] <= 2^31 - 1`.

Example: `nums = [3,4,-1,1]` → `2` (`1` is present, `2` is not). Example: `nums = [7,8,9,11,12]` → `1` (no small positives at all).

hint: The answer for an array of length `n` must lie in `1..n+1`, so any value outside that range is irrelevant.
hint: Use the array itself as a hash table: the value `v` belongs at index `v - 1`.
hint: Repeatedly swap each in-range value into its home slot; afterward the first index `i` whose entry is not `i + 1` reveals the missing positive `i + 1`.

```cpp
// starter
#include <vector>
int firstMissingPositive(std::vector<int>& nums);
```

```cpp
int firstMissingPositive(std::vector<int>& nums) {
    int n = (int)nums.size();
    for (int i = 0; i < n; ++i) {
        while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            std::swap(nums[i], nums[nums[i] - 1]);
        }
    }
    for (int i = 0; i < n; ++i) {
        if (nums[i] != i + 1) return i + 1;
    }
    return n + 1;
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
    { vector<int> n{1,2,0};            if (firstMissingPositive(n) != 3) { std::puts("case1"); return 1; } }
    { vector<int> n{3,4,-1,1};         if (firstMissingPositive(n) != 2) { std::puts("case2"); return 1; } }
    { vector<int> n{7,8,9,11,12};      if (firstMissingPositive(n) != 1) { std::puts("case3"); return 1; } }
    { vector<int> n{1};                if (firstMissingPositive(n) != 2) { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,3};            if (firstMissingPositive(n) != 4) { std::puts("case5"); return 1; } }
    { vector<int> n{2,2,2,2};          if (firstMissingPositive(n) != 1) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because a size-`n` array can miss no positive larger than `n + 1`, we can treat the array as its own hash table where value `v` lives at index `v - 1`. Cyclically swapping each in-range value to its home slot is O(n) amortized (every swap places a value permanently). A final scan returns the first index that does not hold its expected value, or `n + 1` if all slots are filled. O(n) time, O(1) extra space.
