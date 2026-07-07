## challenge: Sliding Window Maximum
tags: sliding-window, monotonic-deque, heap
track: faang
difficulty: hard

Given an array `nums` and a window size `k`, the window slides from left to right one position at a time. Return an array of the maximum value in each window. Aim for O(n) using a monotonic deque of indices.

Constraints: `1 <= nums.length <= 10^5`, `1 <= k <= nums.length`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [1,3,-1,-3,5,3,6,7], k = 3` → `[3,3,5,5,6,7]`. Example: `nums = [1], k = 1` → `[1]`.

hint: The window's maximum only changes when the current max slides out or a larger value enters.
hint: A monotonic deque of indices, values decreasing from front to back, keeps the current maximum at the front.
hint: Before pushing i, pop smaller values off the back (they can never be the max again) and drop front indices that have left the window.

```cpp
// starter
#include <vector>
std::vector<int> maxSlidingWindow(std::vector<int>& nums, int k);
```

```cpp
std::vector<int> maxSlidingWindow(std::vector<int>& nums, int k) {
    std::deque<int> dq;             // indices, values decreasing front->back
    std::vector<int> res;
    for (int i = 0; i < (int)nums.size(); ++i) {
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) res.push_back(nums[dq.front()]);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <deque>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,3,-1,-3,5,3,6,7};
      if (maxSlidingWindow(n, 3) != vector<int>({3,3,5,5,6,7})) { std::puts("case1"); return 1; } }
    { vector<int> n{1};
      if (maxSlidingWindow(n, 1) != vector<int>({1})) { std::puts("case2"); return 1; } }
    { vector<int> n{9,8,7,6};
      if (maxSlidingWindow(n, 2) != vector<int>({9,8,7})) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2,3,4,5};
      if (maxSlidingWindow(n, 5) != vector<int>({5})) { std::puts("case4"); return 1; } }
    { vector<int> n{-7,-8,7,5,7,1,6,0};
      if (maxSlidingWindow(n, 4) != vector<int>({7,7,7,7,7})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A double-ended queue holds indices whose values decrease front-to-back, so its front is always the current window's maximum. Each index enters and leaves the deque exactly once as the window slides. O(n) time, O(k) space.
