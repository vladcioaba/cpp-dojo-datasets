## challenge: Merge Sorted Array
tags: two-pointers, array, sorting
track: faang
difficulty: easy

You are given two integer arrays `nums1` and `nums2` sorted in non-decreasing order, and two integers `m` and `n` giving how many valid elements each holds. `nums1` has length `m + n`, where the first `m` slots hold its elements and the trailing `n` slots are zeros reserved for the merge. Merge `nums2` into `nums1` in place so `nums1` ends up sorted in non-decreasing order.

Constraints: `nums1.length == m + n`, `nums2.length == n`, `0 <= m, n <= 200`, `-10^9 <= nums1[i], nums2[j] <= 10^9`.

Example: `nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3` → `[1,2,2,3,5,6]`. Example: `nums1 = [1], m = 1, nums2 = [], n = 0` → `[1]`.

hint: Merging from the front would overwrite unread elements of `nums1`; the free space is at the back, so fill from there.
hint: Keep a pointer at the last valid element of each array and a write pointer at the very end of `nums1`.
hint: Compare the two tails, copy the larger into the write slot, and step that pointer back; when `nums2` is exhausted the rest of `nums1` is already in place.

```cpp
// starter
#include <vector>
void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n);
```

```cpp
void merge(std::vector<int>& nums1, int m, std::vector<int>& nums2, int n) {
    int i = m - 1, j = n - 1, k = m + n - 1;
    while (j >= 0) {
        if (i >= 0 && nums1[i] > nums2[j]) nums1[k--] = nums1[i--];
        else nums1[k--] = nums2[j--];
    }
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> a{1,2,3,0,0,0}, b{2,5,6}; merge(a,3,b,3); if (a != vector<int>{1,2,2,3,5,6}) { std::puts("case1"); return 1; } }
    { vector<int> a{1}, b{}; merge(a,1,b,0); if (a != vector<int>{1}) { std::puts("case2"); return 1; } }
    { vector<int> a{0}, b{1}; merge(a,0,b,1); if (a != vector<int>{1}) { std::puts("case3"); return 1; } }
    { vector<int> a{4,5,6,0,0,0}, b{1,2,3}; merge(a,3,b,3); if (a != vector<int>{1,2,3,4,5,6}) { std::puts("case4"); return 1; } }
    { vector<int> a{2,0}, b{1}; merge(a,1,b,1); if (a != vector<int>{1,2}) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The empty room in `nums1` sits at its tail, so merge from the back to avoid clobbering values you have not compared yet. Point `i` at the last real element of `nums1`, `j` at the last of `nums2`, and `k` at the final slot. Each step writes the larger of the two tails into position `k` and retreats that pointer. Once `nums2` runs out, any remaining `nums1` prefix is already sorted and correctly placed. O(m + n) time, O(1) extra space.
