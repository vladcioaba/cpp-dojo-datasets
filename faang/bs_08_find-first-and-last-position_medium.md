## challenge: Find First and Last Position of Element in Sorted Array
tags: binary-search, array
track: faang
difficulty: medium

Given an array `nums` sorted in ascending order (possibly with duplicates) and a `target`, return `[first, last]` — the starting and ending indices of `target`. If `target` is not present, return `[-1, -1]`. You must run in O(log n).

Constraints: `0 <= nums.length <= 10^5`, `-10^9 <= nums[i], target <= 10^9`, `nums` is sorted ascending.

Example: `nums = [5,7,7,8,8,10], target = 8` → `[3,4]`. Example: `target = 6` → `[-1,-1]`. Example: `nums = [], target = 0` → `[-1,-1]`.

hint: One binary search only finds *some* occurrence; you need two searches to pin down the two ends of the run.
hint: For the first index, when you hit the target keep looking left (`hi = mid - 1`); for the last index, keep looking right (`lo = mid + 1`), remembering the best match seen.
hint: Handle the empty array and the not-found case up front — both must yield `[-1, -1]`.

```cpp
// starter
#include <vector>
std::vector<int> searchRange(std::vector<int>& nums, int target);
```

```cpp
std::vector<int> searchRange(std::vector<int>& nums, int target) {
    auto bound = [&](bool first) -> int {
        int lo = 0, hi = (int)nums.size() - 1, res = -1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] == target) {
                res = mid;
                if (first) hi = mid - 1;
                else       lo = mid + 1;
            } else if (nums[mid] < target) {
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
        return res;
    };
    return {bound(true), bound(false)};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
static bool eq(const vector<int>& a, int x, int y) { return a.size()==2 && a[0]==x && a[1]==y; }
int main() {
    { vector<int> n{5,7,7,8,8,10}; if (!eq(searchRange(n, 8), 3, 4))   { std::puts("case1"); return 1; } }
    { vector<int> n{5,7,7,8,8,10}; if (!eq(searchRange(n, 6), -1, -1)) { std::puts("case2"); return 1; } }
    { vector<int> n{};             if (!eq(searchRange(n, 0), -1, -1)) { std::puts("case3"); return 1; } }
    { vector<int> n{1};            if (!eq(searchRange(n, 1), 0, 0))   { std::puts("case4"); return 1; } }
    { vector<int> n{2,2};          if (!eq(searchRange(n, 2), 0, 1))   { std::puts("case5"); return 1; } }
    { vector<int> n{1,1,1,1,1};    if (!eq(searchRange(n, 1), 0, 4))   { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Run two boundary-biased binary searches. Both look for the target, but on a match the "first" search continues into the left half while the "last" search continues into the right half, each recording the most recent hit. This locates the two ends of the equal run in O(log n) time, O(1) space, without a linear scan to expand outward.
