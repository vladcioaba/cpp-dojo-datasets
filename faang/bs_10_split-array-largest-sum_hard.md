## challenge: Split Array Largest Sum
tags: binary-search, array, dynamic-programming
track: faang
difficulty: hard

Given an integer array `nums` and an integer `k`, split `nums` into `k` non-empty contiguous subarrays so that the largest subarray sum among them is as small as possible. Return that minimized largest sum.

Constraints: `1 <= nums.length <= 1000`, `0 <= nums[i] <= 10^6`, `1 <= k <= nums.length`.

Example: `nums = [7,2,5,10,8], k = 2` → `18` (split as `[7,2,5]` and `[10,8]`). Example: `nums = [1,2,3,4,5], k = 2` → `9`. Example: `nums = [1,4,4], k = 3` → `4`.

hint: Instead of searching over partitions, search over the *answer*: the largest allowed subarray sum.
hint: For a candidate cap, greedily fill subarrays left to right, cutting whenever the running sum would exceed the cap; the number of pieces needed is monotonic in the cap.
hint: The answer lies between `max(nums)` (each element alone can never be split smaller) and `sum(nums)` (one giant piece). Binary search that range.

```cpp
// starter
#include <vector>
int splitArray(std::vector<int>& nums, int k);
```

```cpp
int splitArray(std::vector<int>& nums, int k) {
    long long lo = 0, hi = 0;
    for (int x : nums) { if (x > lo) lo = x; hi += x; }
    while (lo < hi) {
        long long mid = lo + (hi - lo) / 2;
        int pieces = 1;
        long long cur = 0;
        for (int x : nums) {
            if (cur + x > mid) { ++pieces; cur = x; }
            else cur += x;
        }
        if (pieces <= k) hi = mid;
        else lo = mid + 1;
    }
    return (int)lo;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{7,2,5,10,8}; if (splitArray(n, 2) != 18) { std::puts("case1"); return 1; } }
    { vector<int> n{1,2,3,4,5};  if (splitArray(n, 2) != 9)  { std::puts("case2"); return 1; } }
    { vector<int> n{1,4,4};      if (splitArray(n, 3) != 4)  { std::puts("case3"); return 1; } }
    { vector<int> n{1};          if (splitArray(n, 1) != 1)  { std::puts("case4"); return 1; } }
    { vector<int> n{1,2,3,4,5};  if (splitArray(n, 1) != 15) { std::puts("case5"); return 1; } }
    { vector<int> n{2,3,1,2,4,3};if (splitArray(n, 5) != 4)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Binary search on the answer. The feasibility test "can we split into at most `k` pieces whose sums each stay under cap `C`?" is monotonic in `C`, and checkable greedily in O(n): start a new piece whenever adding the next element would exceed `C`. Search `C` over `[max(nums), sum(nums)]`, driving toward the smallest feasible cap. O(n log(sum)) time, O(1) extra space — far cheaper than the O(n^2 k) dynamic-programming alternative.
