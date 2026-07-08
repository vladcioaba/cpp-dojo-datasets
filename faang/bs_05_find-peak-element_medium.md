## challenge: Find Peak Element
tags: binary-search, array
track: faang
difficulty: medium

A peak element is one that is strictly greater than its neighbors. Given `nums`, return the index of any peak. You may imagine `nums[-1] = nums[n] = -∞`, so the ends can be peaks. Adjacent elements are never equal. You must run in O(log n).

Constraints: `1 <= nums.length <= 1000`, `-2^31 <= nums[i] <= 2^31 - 1`, `nums[i] != nums[i+1]` for all valid `i`.

Example: `nums = [1,2,3,1]` → `2` (value `3` is a peak). Example: `nums = [1,2,1,3,5,6,4]` → `1` or `5` are both accepted. Example: `nums = [1]` → `0`.

hint: You do not need to see the whole array — the slope at the midpoint tells you which side must contain a peak.
hint: If `nums[mid] < nums[mid + 1]`, an ascending step means a peak lies strictly to the right; otherwise a peak lies at `mid` or to its left.
hint: Because the boundaries act like `-∞`, walking uphill can never run off the array — you are guaranteed to converge on a peak.

```cpp
// starter
#include <vector>
int findPeakElement(std::vector<int>& nums);
```

```cpp
int findPeakElement(std::vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < nums[mid + 1]) lo = mid + 1;
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
static bool isPeak(vector<int>& v, int i) {
    bool left  = (i == 0)                 || v[i - 1] < v[i];
    bool right = (i + 1 == (int)v.size()) || v[i + 1] < v[i];
    return i >= 0 && i < (int)v.size() && left && right;
}
int main() {
    { vector<int> n{1,2,3,1};        if (!isPeak(n, findPeakElement(n))) { std::puts("case1"); return 1; } }
    { vector<int> n{1,2,1,3,5,6,4};  if (!isPeak(n, findPeakElement(n))) { std::puts("case2"); return 1; } }
    { vector<int> n{1};              if (!isPeak(n, findPeakElement(n))) { std::puts("case3"); return 1; } }
    { vector<int> n{1,2};            if (!isPeak(n, findPeakElement(n))) { std::puts("case4"); return 1; } }
    { vector<int> n{2,1};            if (!isPeak(n, findPeakElement(n))) { std::puts("case5"); return 1; } }
    { vector<int> n{1,2,3,4,5};      if (!isPeak(n, findPeakElement(n))) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Binary search on the local slope. At `mid`, if the element to the right is larger we are on an ascending stretch, so a peak must exist to the right; otherwise `mid` itself could be a peak or one lies to its left. Since the virtual `-∞` boundaries guarantee the uphill direction always terminates, the window collapses onto a valid peak in O(log n) time, O(1) space.
