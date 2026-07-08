## challenge: Minimum Size Subarray Sum
tags: array, sliding-window
track: faang
difficulty: medium

Given an array of positive integers `nums` and a positive integer `target`, return the minimal length of a contiguous subarray whose sum is greater than or equal to `target`. If no such subarray exists, return `0`.

Constraints: `1 <= target <= 10^9`, `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^4`.

Example: `target = 7, nums = [2,3,1,2,4,3]` → `2` (the subarray `[4,3]`). Example: `target = 4, nums = [1,4,4]` → `1`. Example: `target = 11, nums = [1,1,1,1,1,1,1,1]` → `0`.

hint: Because every value is positive, extending a window only increases its sum and shrinking only decreases it — the sum is monotonic in the window bounds.
hint: Expand the right edge until the window sum reaches `target`, then contract from the left as far as you can while it still qualifies.
hint: Record the window length each time it qualifies during contraction, keeping the smallest.

```cpp
// starter
#include <vector>
int minSubArrayLen(int target, std::vector<int>& nums);
```

```cpp
int minSubArrayLen(int target, std::vector<int>& nums) {
    int left = 0, best = INT_MAX;
    long long sum = 0;
    for (int right = 0; right < (int)nums.size(); ++right) {
        sum += nums[right];
        while (sum >= target) {
            best = std::min(best, right - left + 1);
            sum -= nums[left++];
        }
    }
    return best == INT_MAX ? 0 : best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <climits>
using std::vector;
//__USER__
int main() {
    { vector<int> n{2,3,1,2,4,3}; if (minSubArrayLen(7,n)!=2) { std::puts("case1"); return 1; } }
    { vector<int> n{1,4,4}; if (minSubArrayLen(4,n)!=1) { std::puts("case2"); return 1; } }
    { vector<int> n{1,1,1,1,1,1,1,1}; if (minSubArrayLen(11,n)!=0) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,3,4,5}; if (minSubArrayLen(15,n)!=5) { std::puts("case4"); return 1; } }
    { vector<int> n{5,1,3,5,10,7,4,9,2,8}; if (minSubArrayLen(15,n)!=2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** With all-positive values the window sum grows monotonically as you extend right and drops as you trim left, so a two-pointer window is exact. Add each new element to the running sum; while the sum is at least `target`, record the current window length and remove the leftmost element to try for a shorter qualifying window. Each element is added once and removed once, giving O(n) time and O(1) space. A 64-bit sum guards against overflow.
