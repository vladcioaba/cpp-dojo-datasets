## challenge: Median of Two Sorted Arrays
tags: binary-search, divide-and-conquer, array
track: faang
difficulty: hard

Given two sorted arrays `nums1` and `nums2` of sizes `m` and `n`, return the median of the two sorted arrays combined. The overall run time complexity should be `O(log(m+n))`.

Constraints: `0 <= m, n <= 1000`, `1 <= m + n <= 2000`, `-10^6 <= nums1[i], nums2[i] <= 10^6`, both arrays sorted non-decreasing.

Example: `nums1 = [1,3]`, `nums2 = [2]` -> `2.0`. Example: `nums1 = [1,2]`, `nums2 = [3,4]` -> `2.5`.

hint: A median splits the merged array into a left half and a right half of (almost) equal size where every left element is <= every right element — you only need that partition, not the merge.
hint: Binary search the cut position in the shorter array; the cut in the other array is forced so the two left parts together hold half the elements.
hint: A partition is correct when `maxLeft1 <= minRight2` and `maxLeft2 <= minRight1`; use `INT_MIN`/`INT_MAX` sentinels for cuts at the array ends.

```cpp
// starter
#include <vector>
double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2);
```

```cpp
double findMedianSortedArrays(std::vector<int>& nums1, std::vector<int>& nums2) {
    if (nums1.size() > nums2.size()) return findMedianSortedArrays(nums2, nums1);
    int m = (int)nums1.size(), n = (int)nums2.size();
    int total = m + n, half = (total + 1) / 2;
    int lo = 0, hi = m;
    while (lo <= hi) {
        int i = (lo + hi) / 2;
        int j = half - i;
        int left1  = (i == 0) ? INT_MIN : nums1[i - 1];
        int right1 = (i == m) ? INT_MAX : nums1[i];
        int left2  = (j == 0) ? INT_MIN : nums2[j - 1];
        int right2 = (j == n) ? INT_MAX : nums2[j];
        if (left1 <= right2 && left2 <= right1) {
            if (total % 2 == 1) return std::max(left1, left2);
            return (std::max(left1, left2) + std::min(right1, right2)) / 2.0;
        } else if (left1 > right2) {
            hi = i - 1;
        } else {
            lo = i + 1;
        }
    }
    return 0.0;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
#include <climits>
#include <cmath>
using std::vector;
static bool close(double got, double want) { return std::fabs(got - want) < 1e-6; }
//__USER__
int main() {
    { vector<int> a{1,3},       b{2};     if (!close(findMedianSortedArrays(a, b), 2.0)) { std::puts("case1"); return 1; } }
    { vector<int> a{1,2},       b{3,4};   if (!close(findMedianSortedArrays(a, b), 2.5)) { std::puts("case2"); return 1; } }
    { vector<int> a{},          b{1};     if (!close(findMedianSortedArrays(a, b), 1.0)) { std::puts("case3"); return 1; } }
    { vector<int> a{},          b{2,3};   if (!close(findMedianSortedArrays(a, b), 2.5)) { std::puts("case4"); return 1; } }
    { vector<int> a{1,2,3,4,5}, b{6,7,8}; if (!close(findMedianSortedArrays(a, b), 4.5)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Binary search a partition of the shorter array; the complementary cut in the longer array is fixed so the combined left side holds exactly `(m+n+1)/2` elements. The partition is valid when both cross-boundary inequalities hold, at which point the median comes from the boundary values (`max` of the two left maxima for odd totals, averaged with the `min` of the right minima for even totals). Sentinels handle empty sides. O(log(min(m,n))) time, O(1) space.
