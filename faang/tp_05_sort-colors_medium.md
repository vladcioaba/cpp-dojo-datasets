## challenge: Sort Colors
tags: two-pointers, array, sorting
track: faang
difficulty: medium

Given an array `nums` with `n` objects colored red, white, or blue (encoded as `0`, `1`, and `2`), sort them in place so that objects of the same color are adjacent, in the order red, white, blue. You must solve it without using the library sort, in a single pass with O(1) extra space (the Dutch National Flag problem).

Constraints: `1 <= nums.length <= 300`, `nums[i]` is `0`, `1`, or `2`.

Example: `nums = [2,0,2,1,1,0]` → `[0,0,1,1,2,2]`. Example: `nums = [2,0,1]` → `[0,1,2]`.

hint: With only three distinct values you can partition the array into three regions instead of comparing pairs.
hint: Maintain three pointers: a `low` boundary for the 0s, a `high` boundary for the 2s, and a `mid` scanner in between.
hint: When `mid` sees a 0 swap it down to `low` and advance both; when it sees a 2 swap it up to `high` and shrink `high` (do not advance `mid`); a 1 just advances `mid`.

```cpp
// starter
#include <vector>
void sortColors(std::vector<int>& nums);
```

```cpp
void sortColors(std::vector<int>& nums) {
    int lo = 0, mid = 0, hi = (int)nums.size() - 1;
    while (mid <= hi) {
        if (nums[mid] == 0) {
            std::swap(nums[lo], nums[mid]);
            ++lo;
            ++mid;
        } else if (nums[mid] == 2) {
            std::swap(nums[mid], nums[hi]);
            --hi;
        } else {
            ++mid;
        }
    }
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
    { vector<int> n{2,0,2,1,1,0}; sortColors(n); if (n != vector<int>{0,0,1,1,2,2}) { std::puts("case1"); return 1; } }
    { vector<int> n{2,0,1};       sortColors(n); if (n != vector<int>{0,1,2}) { std::puts("case2"); return 1; } }
    { vector<int> n{0};           sortColors(n); if (n != vector<int>{0}) { std::puts("case3"); return 1; } }
    { vector<int> n{1,0};         sortColors(n); if (n != vector<int>{0,1}) { std::puts("case4"); return 1; } }
    { vector<int> n{2,2,2,1,1,0,0}; sortColors(n); if (n != vector<int>{0,0,1,1,2,2,2}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The Dutch National Flag algorithm keeps three regions using three indices: everything before `low` is 0, everything after `high` is 2, and `mid` scans the unsorted middle. A 0 is swapped into the `low` region (both advance), a 2 is swapped into the `high` region (only `high` retreats, because the swapped-in value is still unexamined), and a 1 stays put while `mid` advances. The scan finishes in a single O(n) pass with O(1) extra space.
