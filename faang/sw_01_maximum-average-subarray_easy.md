## challenge: Maximum Average Subarray I
tags: array, sliding-window
track: faang
difficulty: easy

Given an integer array `nums` and an integer `k`, find a contiguous subarray of length exactly `k` that has the maximum average value, and return that maximum average. Any answer within `10^-5` of the true value is accepted.

Constraints: `1 <= k <= nums.length <= 10^5`, `-10^4 <= nums[i] <= 10^4`.

Example: `nums = [1,12,-5,-6,50,3], k = 4` → `12.75` (the subarray `[12,-5,-6,50]` averages `51/4`). Example: `nums = [5], k = 1` → `5.00000`.

hint: Every candidate subarray has the same length `k`, so comparing sums is enough — divide only once at the end.
hint: Compute the sum of the first `k` elements, then slide: add the incoming element and subtract the outgoing one to get the next window's sum in O(1).
hint: Track the largest window sum seen, then return `bestSum / (double)k`.

```cpp
// starter
#include <vector>
double findMaxAverage(std::vector<int>& nums, int k);
```

```cpp
double findMaxAverage(std::vector<int>& nums, int k) {
    long long sum = 0;
    for (int i = 0; i < k; ++i) sum += nums[i];
    long long best = sum;
    for (int i = k; i < (int)nums.size(); ++i) {
        sum += nums[i] - nums[i - k];
        best = std::max(best, sum);
    }
    return best / (double)k;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <cmath>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,12,-5,-6,50,3}; if (std::fabs(findMaxAverage(n,4) - 12.75) > 1e-5) { std::puts("case1"); return 1; } }
    { vector<int> n{5}; if (std::fabs(findMaxAverage(n,1) - 5.0) > 1e-5) { std::puts("case2"); return 1; } }
    { vector<int> n{0,4,0,3,2}; if (std::fabs(findMaxAverage(n,1) - 4.0) > 1e-5) { std::puts("case3"); return 1; } }
    { vector<int> n{-1,-2,-3,-4}; if (std::fabs(findMaxAverage(n,2) - (-1.5)) > 1e-5) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because all windows share the same length `k`, the one with the maximum average is exactly the one with the maximum sum. Compute the first window's sum, then slide it across the array in O(1) per step by adding the entering element and subtracting the one that fell out. Keep the best sum and divide by `k` once at the end. O(n) time, O(1) space. A 64-bit accumulator avoids overflow on the extreme inputs.
