## challenge: Search in Rotated Sorted Array
tags: binary-search, array
track: faang
difficulty: medium

An ascending sorted array of distinct integers is rotated at an unknown pivot. Given `nums` and a `target`, return its index, or `-1` if absent. You must run in O(log n).

Constraints: `1 <= nums.length <= 5000`, all values distinct, `-10^4 <= nums[i], target <= 10^4`.

Example: `nums = [4,5,6,7,0,1,2], target = 0` → `4`. Example: `target = 3` → `-1`. Example: `nums = [1], target = 0` → `-1`.

hint: Even after rotation, at any midpoint at least one of the two halves is still fully sorted.
hint: Binary search: figure out which half is sorted, then check whether the target lies inside that ordered range.
hint: Compare `nums[lo] <= nums[mid]` to decide which side is sorted, then discard the half that cannot contain the target.

```cpp
// starter
#include <vector>
int search(std::vector<int>& nums, int target);
```

```cpp
int search(std::vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        if (nums[lo] <= nums[mid]) {           // left half sorted
            if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
            else lo = mid + 1;
        } else {                               // right half sorted
            if (nums[mid] < target && target <= nums[hi]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return -1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{4,5,6,7,0,1,2}; if (search(n, 0) != 4)  { std::puts("case1"); return 1; } }
    { vector<int> n{4,5,6,7,0,1,2}; if (search(n, 3) != -1) { std::puts("case2"); return 1; } }
    { vector<int> n{1};             if (search(n, 0) != -1) { std::puts("case3"); return 1; } }
    { vector<int> n{1};             if (search(n, 1) != 0)  { std::puts("case4"); return 1; } }
    { vector<int> n{5,1,3};         if (search(n, 5) != 0)  { std::puts("case5"); return 1; } }
    { vector<int> n{4,5,6,7,0,1,2}; if (search(n, 6) != 2)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A modified binary search. At each step one of [lo, mid] or [mid, hi] is sorted; test whether the target falls within that sorted half's range and narrow accordingly. O(log n) time, O(1) space.
