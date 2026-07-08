## challenge: Remove Duplicates from Sorted Array
tags: two-pointers, array
track: faang
difficulty: easy

Given an integer array `nums` sorted in non-decreasing order, remove the duplicates in place so that each unique element appears only once. The relative order of the elements must be kept the same. Return `k`, the number of unique elements; the first `k` slots of `nums` must hold those unique values (what is left beyond `k` does not matter).

Constraints: `1 <= nums.length <= 3 * 10^4`, `-100 <= nums[i] <= 100`, `nums` is sorted in non-decreasing order.

Example: `nums = [1,1,2]` → `k = 2`, `nums` starts `[1,2,...]`. Example: `nums = [0,0,1,1,1,2,2,3,3,4]` → `k = 5`, `nums` starts `[0,1,2,3,4,...]`.

hint: Because the array is sorted, every group of equal values is contiguous — a duplicate is always adjacent to the value it copies.
hint: Keep a slow write pointer for the position of the next unique slot and a fast read pointer scanning ahead.
hint: Advance the read pointer through the array; whenever `nums[read]` differs from the last written value, copy it to the write pointer and advance the write pointer.

```cpp
// starter
#include <vector>
int removeDuplicates(std::vector<int>& nums);
```

```cpp
int removeDuplicates(std::vector<int>& nums) {
    if (nums.empty()) return 0;
    int k = 1;
    for (int i = 1; i < (int)nums.size(); ++i) {
        if (nums[i] != nums[k - 1]) {
            nums[k] = nums[i];
            ++k;
        }
    }
    return k;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
static bool check(vector<int> nums, int wantK, vector<int> wantPrefix) {
    int k = removeDuplicates(nums);
    if (k != wantK) return false;
    for (int i = 0; i < k; ++i) if (nums[i] != wantPrefix[i]) return false;
    return true;
}
int main() {
    if (!check({1,1,2}, 2, {1,2})) { std::puts("case1"); return 1; }
    if (!check({0,0,1,1,1,2,2,3,3,4}, 5, {0,1,2,3,4})) { std::puts("case2"); return 1; }
    if (!check({1}, 1, {1})) { std::puts("case3"); return 1; }
    if (!check({1,2,3}, 3, {1,2,3})) { std::puts("case4"); return 1; }
    if (!check({5,5,5,5}, 1, {5})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Maintain a slow pointer `k` marking how many unique values have been committed to the front of the array, and a fast pointer scanning every element. Since equal values are adjacent in a sorted array, a new element is unique exactly when it differs from `nums[k-1]`; when that happens, copy it forward and grow `k`. One linear pass with in-place writes gives O(n) time and O(1) extra space.
