## challenge: Check If Array Is Sorted and Rotated
tags: array, arrays-hashing
track: faang
difficulty: medium

Given an array `nums`, return `true` if it was originally sorted in non-decreasing order and then rotated some number of positions (including zero). Otherwise return `false`. There may be duplicates in the original array.

Constraints: `1 <= nums.length <= 100`, `1 <= nums[i] <= 100`.

Example: `nums = [3,4,5,1,2]` → `true` (originally `[1,2,3,4,5]` rotated to start at 3). Example: `nums = [2,1,3,4]` → `false` (no rotation of a sorted array yields this). Example: `nums = [1,2,3]` → `true` (rotated by zero).

hint: A sorted-then-rotated array, viewed circularly, is non-decreasing except at the single "wrap" point where the largest value meets the smallest.
hint: Count the positions `i` where `nums[i] > nums[(i+1) % n]`, comparing each element with its circular successor.
hint: If there is at most one such drop the array is valid; two or more drops mean it could not have come from a single rotation of a sorted array.

```cpp
// starter
#include <vector>
bool check(std::vector<int>& nums);
```

```cpp
bool check(std::vector<int>& nums) {
    int n = (int)nums.size(), drops = 0;
    for (int i = 0; i < n; ++i)
        if (nums[i] > nums[(i + 1) % n]) ++drops;
    return drops <= 1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{3,4,5,1,2}; if (check(n) != true) { std::puts("case1"); return 1; } }
    { vector<int> n{2,1,3,4}; if (check(n) != false) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3}; if (check(n) != true) { std::puts("case3"); return 1; } }
    { vector<int> n{1,1,1}; if (check(n) != true) { std::puts("case4"); return 1; } }
    { vector<int> n{2,1}; if (check(n) != true) { std::puts("case5"); return 1; } }
    { vector<int> n{6,10,6}; if (check(n) != true) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Treat the array as circular and count "drops," positions where an element exceeds its next neighbor (wrapping the last back to the first). A properly sorted array has zero drops; rotating it introduces exactly one drop at the seam where the maximum precedes the minimum. Any array with two or more drops cannot be a single rotation of a non-decreasing sequence, so the test is simply `drops <= 1`. O(n) time, O(1) space.
