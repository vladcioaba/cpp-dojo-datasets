## challenge: Move Zeroes
tags: array, two-pointers, arrays-hashing
track: faang
difficulty: easy

Given an integer array `nums`, shift every `0` to the end of the array while keeping the relative order of the non-zero elements unchanged. Perform the transformation in place without allocating a second array.

Constraints: `1 <= nums.length <= 10^4`, `-2^31 <= nums[i] <= 2^31 - 1`.

Example: `nums = [0,1,0,3,12]` → `[1,3,12,0,0]`. Example: `nums = [0]` → `[0]` (nothing to move).

hint: Think of two positions: one scanning every element, one marking where the next non-zero value belongs.
hint: A slow write pointer trails a fast read pointer; only non-zero values get written forward.
hint: When the reader finds a non-zero value, swap it into the writer's slot and advance the writer — the zeroes drift to the tail automatically.

```cpp
// starter
#include <vector>
void moveZeroes(std::vector<int>& nums);
```

```cpp
void moveZeroes(std::vector<int>& nums) {
    int write = 0;
    for (int read = 0; read < (int)nums.size(); ++read) {
        if (nums[read] != 0) {
            std::swap(nums[read], nums[write]);
            ++write;
        }
    }
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
    { vector<int> n{0,1,0,3,12}; moveZeroes(n); vector<int> w{1,3,12,0,0};
      if (n != w) { std::puts("case1"); return 1; } }
    { vector<int> n{0};          moveZeroes(n); vector<int> w{0};
      if (n != w) { std::puts("case2"); return 1; } }
    { vector<int> n{1,2,3};      moveZeroes(n); vector<int> w{1,2,3};
      if (n != w) { std::puts("case3"); return 1; } }
    { vector<int> n{0,0,1};      moveZeroes(n); vector<int> w{1,0,0};
      if (n != w) { std::puts("case4"); return 1; } }
    { vector<int> n{4,0,5,0,0,6};moveZeroes(n); vector<int> w{4,5,6,0,0,0};
      if (n != w) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A write pointer marks the next landing spot for a non-zero value while a read pointer scans the array. Each non-zero element is swapped into the write slot, preserving order; zeroes are left behind and end up bunched at the tail. Single pass, O(n) time, O(1) extra space.
