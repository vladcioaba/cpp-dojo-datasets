## challenge: Maximum Product Subarray
tags: dynamic-programming, array
track: faang
difficulty: medium

Given an integer array `nums`, find a contiguous non-empty subarray whose element product is the largest, and return that product. The answer is guaranteed to fit in a 32-bit signed integer.

Constraints: `1 <= nums.length <= 2 * 10^4`, `-10 <= nums[i] <= 10`.

Example: `nums = [2,3,-2,4]` → `6` (the subarray `[2,3]`). Example: `nums = [-2,3,-4]` → `24` (the whole array).

hint: Unlike sum, a very negative running product can become the maximum after multiplying by another negative number.
hint: Track both the maximum and the minimum product of subarrays ending at the current index.
hint: When the current value is negative, swap the running max and min before extending, since multiplying flips their roles.

```cpp
// starter
#include <vector>
int maxProduct(std::vector<int>& nums);
```

```cpp
int maxProduct(std::vector<int>& nums) {
    int best = nums[0], curMax = nums[0], curMin = nums[0];
    for (int i = 1; i < (int)nums.size(); ++i) {
        int x = nums[i];
        if (x < 0) std::swap(curMax, curMin);
        curMax = std::max(x, curMax * x);
        curMin = std::min(x, curMin * x);
        best = std::max(best, curMax);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <utility>
using std::vector;
//__USER__
int main() {
    { vector<int> n{2,3,-2,4};      if (maxProduct(n) != 6)  { std::puts("case1"); return 1; } }
    { vector<int> n{-2,0,-1};       if (maxProduct(n) != 0)  { std::puts("case2"); return 1; } }
    { vector<int> n{-2,3,-4};       if (maxProduct(n) != 24) { std::puts("case3"); return 1; } }
    { vector<int> n{2,-5,-2,-4,3};  if (maxProduct(n) != 24) { std::puts("case4"); return 1; } }
    { vector<int> n{-2};            if (maxProduct(n) != -2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because negatives flip sign, track both the largest and smallest products of subarrays ending at each index. On a negative element the roles swap, then each is updated as the better of starting fresh at the current value or extending the previous run. The global answer is the running maximum, computed in O(n) time and O(1) space.
