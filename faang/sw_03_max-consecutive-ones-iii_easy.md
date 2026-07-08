## challenge: Max Consecutive Ones III
tags: array, sliding-window
track: faang
difficulty: easy

Given a binary array `nums` and an integer `k`, return the length of the longest contiguous subarray that contains only `1`s after flipping at most `k` of its `0`s to `1`s.

Constraints: `1 <= nums.length <= 10^5`, `nums[i]` is `0` or `1`, `0 <= k <= nums.length`.

Example: `nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2` → `6` (flip the two zeros at indices 3 and 4... any two of the first three zeros yields six 1s). Example: `nums = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], k = 3` → `10`.

hint: Rephrase the goal: find the longest window that contains at most `k` zeros.
hint: Grow the window to the right; whenever it holds more than `k` zeros, advance the left edge until it holds `k` or fewer again.
hint: The widest valid window reached anywhere during the scan is the answer.

```cpp
// starter
#include <vector>
int longestOnes(std::vector<int>& nums, int k);
```

```cpp
int longestOnes(std::vector<int>& nums, int k) {
    int left = 0, zeros = 0, best = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        if (nums[right] == 0) ++zeros;
        while (zeros > k) {
            if (nums[left] == 0) --zeros;
            ++left;
        }
        best = std::max(best, right - left + 1);
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
    { vector<int> n{1,1,1,0,0,0,1,1,1,1,0}; if (longestOnes(n,2)!=6) { std::puts("case1"); return 1; } }
    { vector<int> n{0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1}; if (longestOnes(n,3)!=10) { std::puts("case2"); return 1; } }
    { vector<int> n{0,0,0}; if (longestOnes(n,0)!=0) { std::puts("case3"); return 1; } }
    { vector<int> n{1,1,1,1}; if (longestOnes(n,0)!=4) { std::puts("case4"); return 1; } }
    { vector<int> n{0,0,1,1,1,0,0}; if (longestOnes(n,1)!=4) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Flipping up to `k` zeros is equivalent to asking for the longest window containing at most `k` zeros. Extend the right boundary and count zeros inside the window; the moment the count exceeds `k`, walk the left boundary forward, decrementing the zero count as zeros leave, until the window is valid again. The maximum width over the whole pass is the answer. Each index enters and leaves the window once, so it is O(n) time and O(1) space.
