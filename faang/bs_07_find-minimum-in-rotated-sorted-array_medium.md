## challenge: Find Minimum in Rotated Sorted Array
tags: binary-search, array
track: faang
difficulty: medium

An ascending sorted array of distinct integers has been rotated between `1` and `n` times at an unknown pivot. Given the resulting array `nums`, return its minimum element. You must run in O(log n).

Constraints: `1 <= nums.length <= 5000`, `-5000 <= nums[i] <= 5000`, all values distinct, and the array is a rotation of an ascending sequence.

Example: `nums = [3,4,5,1,2]` → `1`. Example: `nums = [4,5,6,7,0,1,2]` → `0`. Example: `nums = [11,13,15,17]` → `11`.

hint: The minimum is the single point where the ascending order "wraps around"; everything before it is larger than everything after.
hint: Compare `nums[mid]` with `nums[hi]`: if `nums[mid] > nums[hi]`, the wrap point is strictly to the right of `mid`.
hint: Keep `mid` as a candidate when `nums[mid] <= nums[hi]` (set `hi = mid`), because the minimum could be `mid` itself.

```cpp
// starter
#include <vector>
int findMin(std::vector<int>& nums);
```

```cpp
int findMin(std::vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) lo = mid + 1;
        else hi = mid;
    }
    return nums[lo];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{3,4,5,1,2};     if (findMin(n) != 1)  { std::puts("case1"); return 1; } }
    { vector<int> n{4,5,6,7,0,1,2}; if (findMin(n) != 0)  { std::puts("case2"); return 1; } }
    { vector<int> n{11,13,15,17};   if (findMin(n) != 11) { std::puts("case3"); return 1; } }
    { vector<int> n{2,1};           if (findMin(n) != 1)  { std::puts("case4"); return 1; } }
    { vector<int> n{1};             if (findMin(n) != 1)  { std::puts("case5"); return 1; } }
    { vector<int> n{5,1,2,3,4};     if (findMin(n) != 1)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Compare the midpoint to the right endpoint rather than the left, which cleanly identifies which half holds the rotation point. If `nums[mid] > nums[hi]`, the array wraps somewhere after `mid`, so search right; otherwise the minimum is at `mid` or to its left. Comparing against `nums[hi]` sidesteps the ambiguity that arises when comparing against `nums[lo]` on an already-sorted segment. O(log n) time, O(1) space.
