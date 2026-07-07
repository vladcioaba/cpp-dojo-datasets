## challenge: Container With Most Water
tags: two-pointers, greedy, array
track: faang
difficulty: medium

Given `n` non-negative integers `height` where each represents a vertical line at coordinate `i` of height `height[i]`, find two lines that together with the x-axis form a container holding the most water. Return the maximum area. You may not slant the container.

Constraints: `2 <= height.length <= 10^5`, `0 <= height[i] <= 10^4`.

Example: `height = [1,8,6,2,5,4,8,3,7]` → `49`. Example: `height = [1,1]` → `1`.

hint: Area is the width times the shorter of the two walls, so begin as wide as possible.
hint: Use two pointers at the ends; moving the taller wall inward can never help, so always move the shorter one.

```cpp
// starter
#include <vector>
int maxArea(std::vector<int>& height);
```

```cpp
int maxArea(std::vector<int>& height) {
    int i = 0, j = (int)height.size() - 1, best = 0;
    while (i < j) {
        int h = std::min(height[i], height[j]);
        best = std::max(best, h * (j - i));
        if (height[i] < height[j]) ++i;
        else --j;
    }
    return best;
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
    { vector<int> h{1,8,6,2,5,4,8,3,7}; if (maxArea(h) != 49) { std::puts("case1"); return 1; } }
    { vector<int> h{1,1};               if (maxArea(h) != 1)  { std::puts("case2"); return 1; } }
    { vector<int> h{4,3,2,1,4};         if (maxArea(h) != 16) { std::puts("case3"); return 1; } }
    { vector<int> h{1,2,1};             if (maxArea(h) != 2)  { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** With pointers at both ends the area is `min(height[i], height[j]) * (j - i)`. Moving the shorter wall inward is the only move that can raise the limiting height, so greedily advance it. O(n) time, O(1) space.
