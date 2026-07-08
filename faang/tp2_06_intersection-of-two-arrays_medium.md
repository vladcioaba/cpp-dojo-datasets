## challenge: Intersection of Two Arrays
tags: two-pointers, array, sorting, hash-table
track: faang
difficulty: medium

Given two integer arrays `nums1` and `nums2`, return an array of their intersection: each element in the result must be unique, and you may return the values in any order.

Constraints: `1 <= nums1.length, nums2.length <= 1000`, `0 <= nums1[i], nums2[j] <= 1000`.

Example: `nums1 = [1,2,2,1], nums2 = [2,2]` → `[2]`. Example: `nums1 = [4,9,5], nums2 = [9,4,9,8,4]` → `[4,9]` (order may vary).

hint: If both arrays are sorted, a shared value can be found by walking them together like a merge.
hint: Move the pointer that references the smaller value; equality means you found a common element.
hint: To keep the result unique, skip appending a value that equals the one you just recorded, and advance both pointers on a match.

```cpp
// starter
#include <vector>
std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2);
```

```cpp
std::vector<int> intersection(std::vector<int>& nums1, std::vector<int>& nums2) {
    std::sort(nums1.begin(), nums1.end());
    std::sort(nums2.begin(), nums2.end());
    std::vector<int> res;
    int i = 0, j = 0;
    int n = (int)nums1.size(), m = (int)nums2.size();
    while (i < n && j < m) {
        if (nums1[i] < nums2[j]) ++i;
        else if (nums1[i] > nums2[j]) ++j;
        else {
            if (res.empty() || res.back() != nums1[i]) res.push_back(nums1[i]);
            ++i; ++j;
        }
    }
    return res;
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
    { vector<int> a{1,2,2,1}, b{2,2}; auto r = intersection(a,b); std::sort(r.begin(),r.end()); if (r != vector<int>{2}) { std::puts("case1"); return 1; } }
    { vector<int> a{4,9,5}, b{9,4,9,8,4}; auto r = intersection(a,b); std::sort(r.begin(),r.end()); if (r != vector<int>{4,9}) { std::puts("case2"); return 1; } }
    { vector<int> a{1,2,3}, b{4,5,6}; auto r = intersection(a,b); std::sort(r.begin(),r.end()); if (r != vector<int>{}) { std::puts("case3"); return 1; } }
    { vector<int> a{1}, b{1}; auto r = intersection(a,b); std::sort(r.begin(),r.end()); if (r != vector<int>{1}) { std::puts("case4"); return 1; } }
    { vector<int> a{3,1,2}, b{2,3,3}; auto r = intersection(a,b); std::sort(r.begin(),r.end()); if (r != vector<int>{2,3}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort both arrays, then sweep with a pointer in each. Whichever pointer references the smaller value advances, since that value cannot appear later in the other array. When the two referenced values are equal you have a common element: record it only if it differs from the last one appended (guaranteeing uniqueness), then step both pointers past it. Sorting dominates at O(n log n + m log m) time with O(1) extra space beyond the output.
