## challenge: Sliding Window Median
tags: array, sliding-window, heap, ordered-set
track: faang
difficulty: hard

The median of a list is its middle value when sorted, or the average of the two middle values when the length is even. Given an integer array `nums` and a window size `k`, the window slides one position at a time from the far left to the far right. Return an array of the medians of every window of size `k`, in order.

Constraints: `1 <= k <= nums.length <= 10^5`, `-2^31 <= nums[i] <= 2^31 - 1`. Any answer within `10^-5` of the true median is accepted.

Example: `nums = [1,3,-1,-3,5,3,6,7], k = 3` → `[1,-1,-1,3,5,6]`. Example: `nums = [1,4,2,3], k = 4` → `[2.5]`. Example: `nums = [2,3,4], k = 1` → `[2,3,4]`.

hint: You need a structure that stays sorted while supporting insertion and deletion — a balanced multiset fits, keeping an iterator parked at the median.

hint: When the window advances, insert the entering value and erase the leaving one, then nudge the median iterator left or right by one depending on whether each change happened below or above it.

hint: Averaging the two central values of an even window can overflow 32-bit arithmetic near the integer limits, so promote to `double` before adding.

```cpp
// starter
#include <vector>
std::vector<double> medianSlidingWindow(std::vector<int>& nums, int k);
```

```cpp
std::vector<double> medianSlidingWindow(std::vector<int>& nums, int k) {
    std::multiset<int> window(nums.begin(), nums.begin() + k);
    auto mid = std::next(window.begin(), (k - 1) / 2);
    std::vector<double> ans;
    for (int i = k;; ++i) {
        ans.push_back(((double)*mid + *std::next(mid, 1 - (k & 1))) / 2.0);
        if (i == (int)nums.size()) return ans;
        window.insert(nums[i]);
        if (nums[i] < *mid) --mid;
        if (nums[i - k] <= *mid) ++mid;
        window.erase(window.lower_bound(nums[i - k]));
    }
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <set>
#include <iterator>
#include <cmath>
using std::vector;
static bool eq(const vector<double>& a, const vector<double>& b) {
    if (a.size() != b.size()) return false;
    for (size_t i = 0; i < a.size(); ++i) if (std::fabs(a[i] - b[i]) > 1e-5) return false;
    return true;
}
//__USER__
int main() {
    { vector<int> n{1,3,-1,-3,5,3,6,7}; vector<double> e{1,-1,-1,3,5,6};
      if (!eq(medianSlidingWindow(n,3), e)) { std::puts("case1"); return 1; } }
    { vector<int> n{1,4,2,3}; vector<double> e{2.5};
      if (!eq(medianSlidingWindow(n,4), e)) { std::puts("case2"); return 1; } }
    { vector<int> n{2,3,4}; vector<double> e{2,3,4};
      if (!eq(medianSlidingWindow(n,1), e)) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2}; vector<double> e{1.5};
      if (!eq(medianSlidingWindow(n,2), e)) { std::puts("case4"); return 1; } }
    { vector<int> n{2147483647,2147483647}; vector<double> e{2147483647.0};
      if (!eq(medianSlidingWindow(n,2), e)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Maintain the window as an ordered multiset and keep an iterator `mid` on the lower of the two central positions. Reading the median is O(1): for an odd `k` it is `*mid`, for an even `k` it is the average of `*mid` and its successor — the expression `std::next(mid, 1 - (k & 1))` selects the right partner in both cases. Sliding one step means one insertion and one deletion; adjust `mid` by comparing each changed value against `*mid` so it always lands on the correct central slot. Insert/erase/lower_bound are O(log k), so the whole scan is O(n log k). Promoting the two middle values to `double` before averaging avoids 32-bit overflow at the extremes.
