## challenge: Longest Subarray of 1's After Deleting One Element
tags: array, sliding-window
track: faang
difficulty: medium

Given a binary array `nums`, you must delete exactly one element. Return the length of the longest contiguous subarray containing only `1`s in the resulting array. If deleting the one element leaves no `1`s, return `0`.

Constraints: `1 <= nums.length <= 10^5`, `nums[i]` is `0` or `1`.

Example: `nums = [1,1,0,1]` → `3` (delete the `0`). Example: `nums = [0,1,1,1,0,1,1,0,1]` → `5`. Example: `nums = [1,1,1]` → `2` (one element must be deleted even if all are `1`).

hint: Deleting one element means the winning window may contain at most one `0`, which you picture removing.
hint: Slide a window that holds at most one zero; when a second zero enters, advance the left edge past the first.
hint: Because exactly one element is always removed, the answer is the widest valid window's width minus one.

```cpp
// starter
#include <vector>
int longestSubarray(std::vector<int>& nums);
```

```cpp
int longestSubarray(std::vector<int>& nums) {
    int left = 0, zeros = 0, best = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        if (nums[right] == 0) ++zeros;
        while (zeros > 1) {
            if (nums[left] == 0) --zeros;
            ++left;
        }
        best = std::max(best, right - left);
    }
    return best;
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
    { vector<int> n{1,1,0,1}; if (longestSubarray(n)!=3) { std::puts("case1"); return 1; } }
    { vector<int> n{0,1,1,1,0,1,1,0,1}; if (longestSubarray(n)!=5) { std::puts("case2"); return 1; } }
    { vector<int> n{1,1,1}; if (longestSubarray(n)!=2) { std::puts("case3"); return 1; } }
    { vector<int> n{0,0,0}; if (longestSubarray(n)!=0) { std::puts("case4"); return 1; } }
    { vector<int> n{1,1,0,0,1,1,1,0,1}; if (longestSubarray(n)!=4) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Since exactly one element is deleted, the best result is the longest window that contains at most one zero (the zero being the element you delete). Slide a window over the array, counting zeros; when a second zero enters, move the left edge forward until only one zero remains. Record `right - left` — that is the window width minus one, which already accounts for the mandatory deletion and correctly yields `n - 1` for an all-ones array. O(n) time, O(1) space.
