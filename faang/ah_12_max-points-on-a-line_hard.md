## challenge: Max Points on a Line
tags: array, hash-table, math, geometry, arrays-hashing
track: faang
difficulty: hard

Given an array `points` where `points[i] = [xi, yi]` marks a distinct point on the 2D plane, return the maximum number of points that all lie on a single straight line.

Constraints: `1 <= points.length <= 300`, `-10^4 <= xi, yi <= 10^4`, all points are distinct.

Example: `points = [[1,1],[2,2],[3,3]]` → `3` (all collinear on `y = x`). Example: `points = [[1,1],[3,2],[5,3],[4,1],[2,3],[1,4]]` → `4` (the points `[1,4],[2,3],[3,2],[4,1]` lie on `x + y = 5`).

hint: Fix one anchor point; every other point defines a slope with it, and points sharing a slope through the anchor are collinear with it.
hint: Represent each slope exactly (avoid floating point) by reducing the direction vector `(dx, dy)` with the gcd and normalizing its sign.
hint: For each anchor, count identical reduced slopes in a hash map; the best count plus the anchor itself is a line through that anchor — take the maximum over all anchors.

```cpp
// starter
#include <vector>
int maxPoints(std::vector<std::vector<int>>& points);
```

```cpp
int maxPoints(std::vector<std::vector<int>>& points) {
    int n = (int)points.size();
    if (n <= 2) return n;
    int best = 1;
    for (int i = 0; i < n; ++i) {
        std::unordered_map<long long, int> slopeCount;
        int localBest = 0;
        for (int j = i + 1; j < n; ++j) {
            int dx = points[j][0] - points[i][0];
            int dy = points[j][1] - points[i][1];
            int g = std::gcd(dx, dy);
            if (g != 0) { dx /= g; dy /= g; }
            if (dx < 0 || (dx == 0 && dy < 0)) { dx = -dx; dy = -dy; }
            long long key = (long long)dx * 100003LL + dy;
            localBest = std::max(localBest, ++slopeCount[key]);
        }
        best = std::max(best, localBest + 1);
    }
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <numeric>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> p{{1,1},{2,2},{3,3}};                             if (maxPoints(p) != 3) { std::puts("case1"); return 1; } }
    { vector<vector<int>> p{{1,1},{3,2},{5,3},{4,1},{2,3},{1,4}};           if (maxPoints(p) != 4) { std::puts("case2"); return 1; } }
    { vector<vector<int>> p{{0,0}};                                        if (maxPoints(p) != 1) { std::puts("case3"); return 1; } }
    { vector<vector<int>> p{{0,0},{1,1}};                                  if (maxPoints(p) != 2) { std::puts("case4"); return 1; } }
    { vector<vector<int>> p{{2,3},{3,3},{-5,3}};                           if (maxPoints(p) != 3) { std::puts("case5"); return 1; } }
    { vector<vector<int>> p{{1,1},{1,2},{1,3},{2,2}};                      if (maxPoints(p) != 3) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Anchor on each point in turn and bucket every other point by the slope of the segment joining them. To compare slopes exactly, reduce the direction vector `(dx, dy)` by its gcd and force a canonical sign, then use it as a hash key. The largest bucket for an anchor, plus the anchor itself, is the biggest line through it; the global maximum answers the problem. With `n` anchors each doing O(n) work, this is O(n^2) time and O(n) space.
