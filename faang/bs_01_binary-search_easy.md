## challenge: Binary Search
tags: binary-search, array
track: faang
difficulty: easy

Given a sorted (ascending) array of distinct integers `nums` and a `target`, return the index of `target` if it is present, or `-1` if it is not. You must write an algorithm with O(log n) runtime.

Constraints: `1 <= nums.length <= 10^4`, `-10^4 <= nums[i], target <= 10^4`, all values are distinct and sorted ascending.

Example: `nums = [-1,0,3,5,9,12], target = 9` → `4`. Example: `target = 2` → `-1`. Example: `nums = [5], target = 5` → `0`.

hint: Because the array is sorted, comparing the middle element to the target tells you which half can possibly contain it.
hint: Keep a `[lo, hi]` window of candidate indices; each comparison halves it, giving O(log n) steps.
hint: Watch the loop boundary — use `lo <= hi` with an inclusive `hi = size - 1`, and move past `mid` with `mid + 1` / `mid - 1` so you never loop forever.

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
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
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
    { vector<int> n{-1,0,3,5,9,12}; if (search(n, 9) != 4)  { std::puts("case1"); return 1; } }
    { vector<int> n{-1,0,3,5,9,12}; if (search(n, 2) != -1) { std::puts("case2"); return 1; } }
    { vector<int> n{5};             if (search(n, 5) != 0)  { std::puts("case3"); return 1; } }
    { vector<int> n{5};             if (search(n, -5) != -1){ std::puts("case4"); return 1; } }
    { vector<int> n{2,5};           if (search(n, 5) != 1)  { std::puts("case5"); return 1; } }
    { vector<int> n{-10,-3,0,4,8};  if (search(n, -10) != 0){ std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Textbook binary search. Maintain an inclusive candidate window `[lo, hi]`; compare `nums[mid]` to the target and discard the half that cannot hold it. Each iteration halves the search space, so the algorithm runs in O(log n) time and O(1) space. Computing `mid = lo + (hi - lo) / 2` avoids the classic `lo + hi` integer overflow.
