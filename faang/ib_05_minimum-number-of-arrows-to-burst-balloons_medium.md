## challenge: Minimum Number of Arrows to Burst Balloons
tags: intervals, greedy, sorting
track: faang
difficulty: medium

There are balloons taped to a wall, each given as horizontal interval `points[i] = [x_start, x_end]`. An arrow shot straight up at coordinate `x` bursts every balloon whose interval satisfies `x_start <= x <= x_end`. Return the minimum number of arrows needed to burst all balloons.

Constraints: `1 <= points.length <= 10^5`, `points[i].length == 2`, `-2^31 <= x_start <= x_end <= 2^31 - 1`.

Example: `points = [[10,16],[2,8],[1,6],[7,12]]` → `2`. Example: `points = [[1,2],[3,4],[5,6],[7,8]]` → `4`. Example: `points = [[1,2],[2,3],[3,4],[4,5]]` → `2`.

hint: One arrow can pop every balloon whose intervals share a common point. So you want to cover all intervals with the fewest common-point groups.
hint: Sort by end coordinate. Fire an arrow at the end of the first balloon; it also pops any later balloon that starts at or before that coordinate.
hint: Keep the position of the last arrow. When the next balloon's start exceeds it, you need a new arrow placed at that balloon's end.

```cpp
// starter
#include <vector>
int findMinArrowShots(std::vector<std::vector<int>>& points);
```

```cpp
int findMinArrowShots(std::vector<std::vector<int>>& points) {
    if (points.empty()) return 0;
    std::sort(points.begin(), points.end(),
              [](const std::vector<int>& a, const std::vector<int>& b) { return a[1] < b[1]; });
    int arrows = 1;
    long long arrowAt = points[0][1];
    for (int i = 1; i < (int)points.size(); ++i) {
        if (points[i][0] > arrowAt) {
            ++arrows;
            arrowAt = points[i][1];
        }
    }
    return arrows;
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
    { vector<vector<int>> p{{10,16},{2,8},{1,6},{7,12}};
      if (findMinArrowShots(p) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> p{{1,2},{3,4},{5,6},{7,8}};
      if (findMinArrowShots(p) != 4) { std::puts("case2"); return 1; } }
    { vector<vector<int>> p{{1,2},{2,3},{3,4},{4,5}};
      if (findMinArrowShots(p) != 2) { std::puts("case3"); return 1; } }
    { vector<vector<int>> p{{2,3},{2,3}};
      if (findMinArrowShots(p) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> p{{-2147483648,2147483647}};
      if (findMinArrowShots(p) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Bursting all balloons with the fewest arrows is an interval-covering problem. Sort balloons by their right edge; place the first arrow at that edge, which greedily pops every subsequent balloon whose start lies at or before it. Only when a balloon starts strictly beyond the current arrow do we need a fresh arrow, positioned at the new balloon's end. Comparing starts against the arrow position with a 64-bit variable avoids overflow near the `int` limits. O(n log n) time, O(1) extra space.
