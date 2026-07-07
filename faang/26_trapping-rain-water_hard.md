## challenge: Trapping Rain Water
tags: two-pointers, dynamic-programming, stack
track: faang
difficulty: hard

Given `n` non-negative integers `height` representing an elevation map where each bar has width `1`, compute how much water it can trap after raining. Solve it in O(n) time and O(1) extra space with two pointers.

Constraints: `1 <= height.length <= 2*10^4`, `0 <= height[i] <= 10^5`.

Example: `height = [0,1,0,2,1,0,1,3,2,1,2,1]` → `6`. Example: `height = [4,2,0,3,2,5]` → `9`.

hint: Water sitting above a bar is bounded by the shorter of the tallest wall to its left and the tallest to its right.
hint: Two pointers from both ends, tracking leftMax and rightMax, advancing the side with the smaller wall.
hint: The side with the smaller running max is safe to settle, because the taller opposite side guarantees the bound holds.

```cpp
// starter
#include <vector>
int trap(std::vector<int>& height);
```

```cpp
int trap(std::vector<int>& height) {
    int l = 0, r = (int)height.size() - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (height[l] < height[r]) {
            leftMax = std::max(leftMax, height[l]);
            water += leftMax - height[l];
            ++l;
        } else {
            rightMax = std::max(rightMax, height[r]);
            water += rightMax - height[r];
            --r;
        }
    }
    return water;
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
    { vector<int> h{0,1,0,2,1,0,1,3,2,1,2,1}; if (trap(h) != 6) { std::puts("case1"); return 1; } }
    { vector<int> h{4,2,0,3,2,5};             if (trap(h) != 9) { std::puts("case2"); return 1; } }
    { vector<int> h{1,2,3,4,5};               if (trap(h) != 0) { std::puts("case3"); return 1; } }
    { vector<int> h{5};                       if (trap(h) != 0) { std::puts("case4"); return 1; } }
    { vector<int> h{3,0,3};                   if (trap(h) != 3) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Two pointers move inward; the side with the lower running maximum determines the water trapped there, because the opposite side is known to be at least as tall. Add `runningMax - height` at each step. O(n) time, O(1) space.
