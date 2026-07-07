## challenge: Largest Rectangle in Histogram
tags: monotonic-stack, stack, array
track: faang
difficulty: hard

Given an array `heights` representing the bar heights of a histogram where each bar has width `1`, return the area of the largest rectangle that fits entirely within the histogram.

Constraints: `0 <= heights.length <= 10^5`, `0 <= heights[i] <= 10^4`.

Example: `heights = [2,1,5,6,2,3]` -> `10` (the bars of height 5 and 6 span width 2). Example: `heights = [2,4]` -> `4`.

hint: For each bar the widest rectangle capped at its height extends left and right until it meets a strictly shorter bar.
hint: A monotonic increasing stack of indices lets you find, in O(n), the previous and next shorter bar for every bar.
hint: When you pop an index, the popped bar's height is the limiting height and the width spans from the new stack top (exclusive) to the current index (exclusive); append a sentinel height of `0` to flush the stack.

```cpp
// starter
#include <vector>
int largestRectangleArea(std::vector<int>& heights);
```

```cpp
int largestRectangleArea(std::vector<int>& heights) {
    std::stack<int> st; // indices with strictly increasing heights
    int best = 0;
    int n = (int)heights.size();
    for (int i = 0; i <= n; ++i) {
        int h = (i == n) ? 0 : heights[i];
        while (!st.empty() && heights[st.top()] >= h) {
            int height = heights[st.top()]; st.pop();
            int width = st.empty() ? i : i - st.top() - 1;
            best = std::max(best, height * width);
        }
        st.push(i);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <stack>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> h{2,1,5,6,2,3}; if (largestRectangleArea(h) != 10) { std::puts("case1"); return 1; } }
    { vector<int> h{2,4};         if (largestRectangleArea(h) != 4)  { std::puts("case2"); return 1; } }
    { vector<int> h{1};           if (largestRectangleArea(h) != 1)  { std::puts("case3"); return 1; } }
    { vector<int> h{0};           if (largestRectangleArea(h) != 0)  { std::puts("case4"); return 1; } }
    { vector<int> h{};            if (largestRectangleArea(h) != 0)  { std::puts("case5"); return 1; } }
    { vector<int> h{2,1,2};       if (largestRectangleArea(h) != 3)  { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sweep left to right maintaining a stack of indices whose heights are increasing. When the current bar is not taller than the stack top, pop it: the popped height is the rectangle's height and its width runs from the element now below the top (exclusive) up to the current index (exclusive). A trailing sentinel of height `0` forces every remaining bar to be resolved. Each index is pushed and popped once, giving O(n) time and O(n) space.
