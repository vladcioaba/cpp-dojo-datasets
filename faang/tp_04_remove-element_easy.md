## challenge: Remove Element
tags: two-pointers, array
track: faang
difficulty: easy

Given an integer array `nums` and an integer `val`, remove all occurrences of `val` in place. Return `k`, the number of elements that are not equal to `val`; the first `k` slots of `nums` must contain the kept elements (in any order). Anything past index `k` is ignored.

Constraints: `0 <= nums.length <= 100`, `0 <= nums[i] <= 50`, `0 <= val <= 100`.

Example: `nums = [3,2,2,3], val = 3` → `k = 2`, first two slots are `{2,2}`. Example: `nums = [0,1,2,2,3,0,4,2], val = 2` → `k = 5`, first five slots are `{0,1,3,0,4}` in some order.

hint: You only care about the elements you keep; the ones equal to `val` can simply be overwritten.
hint: A slow write pointer marks the next kept slot while a fast read pointer scans every element.
hint: For each read position, if `nums[read] != val` copy it to the write pointer and advance the write pointer; otherwise skip it.

```cpp
// starter
#include <vector>
int removeElement(std::vector<int>& nums, int val);
```

```cpp
int removeElement(std::vector<int>& nums, int val) {
    int k = 0;
    for (int i = 0; i < (int)nums.size(); ++i) {
        if (nums[i] != val) {
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
#include <algorithm>
using std::vector;
//__USER__
static bool check(vector<int> nums, int val, int wantK, vector<int> wantKept) {
    int k = removeElement(nums, val);
    if (k != wantK) return false;
    vector<int> got(nums.begin(), nums.begin() + k);
    std::sort(got.begin(), got.end());
    std::sort(wantKept.begin(), wantKept.end());
    return got == wantKept;
}
int main() {
    if (!check({3,2,2,3}, 3, 2, {2,2})) { std::puts("case1"); return 1; }
    if (!check({0,1,2,2,3,0,4,2}, 2, 5, {0,1,3,0,4})) { std::puts("case2"); return 1; }
    if (!check({1}, 1, 0, {})) { std::puts("case3"); return 1; }
    if (!check({4,5}, 6, 2, {4,5})) { std::puts("case4"); return 1; }
    if (!check({}, 0, 0, {})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Treat the array as its own output buffer. A slow pointer `k` records how many kept elements have been placed at the front, while a fast pointer scans the whole array. Every element that is not `val` gets copied to slot `k`, which then advances; elements equal to `val` are simply passed over. One linear scan gives O(n) time and O(1) extra space.
