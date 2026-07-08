## challenge: Smallest Range Covering Elements from K Lists
tags: heap, priority-queue, sliding-window, merging

track: faang
difficulty: hard

You have `k` lists of integers, each sorted in non-decreasing order. Find the smallest range `[a, b]` that includes at least one number from each of the `k` lists. Range `[a, b]` is smaller than `[c, d]` if `b - a < d - c`, or `b - a == d - c` and `a < c`.

Constraints: `1 <= k <= 3500`, `1 <= nums[i].length <= 50`, `-10^5 <= nums[i][j] <= 10^5`, each list is sorted ascending.

Example: `nums = [[4,10,15,24,26],[0,9,12,20],[5,18,22,30]]` → `[20,24]`. Example: `nums = [[1,2,3],[1,2,3],[1,2,3]]` → `[1,1]`. Example: `nums = [[10],[11]]` → `[10,11]`.

hint: A valid range must span from the current minimum across the picked elements up to the current maximum, one pick per list.
hint: Keep one "cursor" element per list; the range is `[min cursor, max cursor]`. To shrink it, advance the list holding the minimum.
hint: A min-heap over the cursors gives the minimum in O(log k); track the running maximum separately, and stop when any list's cursor can't advance.

```cpp
// starter
#include <vector>
std::vector<int> smallestRange(std::vector<std::vector<int>>& nums);
```

```cpp
std::vector<int> smallestRange(std::vector<std::vector<int>>& nums) {
    typedef std::tuple<int, int, int> T;   // (value, list index, element index)
    std::priority_queue<T, std::vector<T>, std::greater<T>> pq;
    int curMax = INT_MIN;
    for (int i = 0; i < (int)nums.size(); ++i) {
        pq.push({nums[i][0], i, 0});
        curMax = std::max(curMax, nums[i][0]);
    }
    int bestLo = 0, bestHi = 0;
    bool have = false;
    while (true) {
        auto [val, li, ei] = pq.top(); pq.pop();
        if (!have || curMax - val < bestHi - bestLo) {
            have = true; bestLo = val; bestHi = curMax;
        }
        if (ei + 1 == (int)nums[li].size()) break;   // this list is exhausted
        int nv = nums[li][ei + 1];
        curMax = std::max(curMax, nv);
        pq.push({nv, li, ei + 1});
    }
    return {bestLo, bestHi};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <tuple>
#include <climits>
#include <algorithm>
#include <functional>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> n{{4,10,15,24,26},{0,9,12,20},{5,18,22,30}};
      if (smallestRange(n) != vector<int>({20,24})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> n{{1,2,3},{1,2,3},{1,2,3}};
      if (smallestRange(n) != vector<int>({1,1}))   { std::puts("case2"); return 1; } }
    { vector<vector<int>> n{{10},{11}};
      if (smallestRange(n) != vector<int>({10,11})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> n{{1},{2},{3}};
      if (smallestRange(n) != vector<int>({1,3}))   { std::puts("case4"); return 1; } }
    { vector<vector<int>> n{{-1,0,1}};
      if (smallestRange(n) != vector<int>({-1,-1})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Maintain one cursor per list; the elements under the cursors always contain exactly one number from each list, and the covering range is `[min cursor, max cursor]`. A min-heap over the cursors exposes the minimum, while a separate variable tracks the running maximum. Each step records the current range if it's the best so far, then advances the list holding the minimum — the only move that can possibly shrink the range. When that list has no next element, no smaller range is reachable, so stop. Every element is pushed once, giving O(N log k) time for `N` total elements and O(k) heap space.
