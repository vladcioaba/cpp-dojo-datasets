## challenge: Search Insert Position
tags: binary-search, array
track: faang
difficulty: easy

Given a sorted array of distinct integers `nums` and a `target`, return the index where `target` is found. If it is absent, return the index where it would be inserted to keep the array sorted. You must run in O(log n).

Constraints: `1 <= nums.length <= 10^4`, `-10^4 <= nums[i], target <= 10^4`, `nums` is sorted ascending with distinct values.

Example: `nums = [1,3,5,6], target = 5` → `2`. Example: `target = 2` → `1`. Example: `target = 7` → `4`. Example: `target = 0` → `0`.

hint: The answer is the number of elements strictly less than `target` — this is a lower-bound search.
hint: Use a half-open window: `lo = 0`, `hi = n` (one past the end), and loop while `lo < hi`. `hi` can legally become the array length for an insert at the end.
hint: When `nums[mid] < target`, the insertion point is to the right (`lo = mid + 1`); otherwise `mid` is still a candidate (`hi = mid`).

```cpp
// starter
#include <vector>
int searchInsert(std::vector<int>& nums, int target);
```

```cpp
int searchInsert(std::vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,3,5,6}; if (searchInsert(n, 5) != 2) { std::puts("case1"); return 1; } }
    { vector<int> n{1,3,5,6}; if (searchInsert(n, 2) != 1) { std::puts("case2"); return 1; } }
    { vector<int> n{1,3,5,6}; if (searchInsert(n, 7) != 4) { std::puts("case3"); return 1; } }
    { vector<int> n{1,3,5,6}; if (searchInsert(n, 0) != 0) { std::puts("case4"); return 1; } }
    { vector<int> n{1};       if (searchInsert(n, 0) != 0) { std::puts("case5"); return 1; } }
    { vector<int> n{1};       if (searchInsert(n, 2) != 1) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** This is `std::lower_bound` written by hand. The insertion index equals the count of elements less than `target`. A half-open window `[lo, hi)` with `hi` initialized to `n` naturally allows the answer to be the array length (append at the end). O(log n) time, O(1) space.
