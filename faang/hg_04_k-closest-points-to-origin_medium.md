## challenge: K Closest Points to Origin
tags: heap, priority-queue, sorting, math
track: faang
difficulty: medium

Given an array of `points` where `points[i] = [xi, yi]` on the plane and an integer `k`, return the `k` points closest to the origin `(0, 0)` by Euclidean distance. The answer is guaranteed to be unique (up to order) and may be returned in any order.

Constraints: `1 <= k <= points.length <= 10^5`, `-10^4 <= xi, yi <= 10^4`.

Example: `points = [[1,3],[-2,2]], k = 1` → `[[-2,2]]`. Example: `points = [[3,3],[5,-1],[-2,4]], k = 2` → `[[3,3],[-2,4]]`.

hint: Comparing squared distances `x*x + y*y` avoids square roots and keeps everything integral.
hint: You only need the k smallest distances, not a full ordering — keep a size-k max-heap keyed on distance.
hint: When the heap exceeds size k, pop its largest so it always holds the current k closest.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> kClosest(std::vector<std::vector<int>>& points, int k);
```

```cpp
std::vector<std::vector<int>> kClosest(std::vector<std::vector<int>>& points, int k) {
    std::priority_queue<std::pair<long long, int>> pq;
    for (int i = 0; i < (int)points.size(); ++i) {
        long long d = (long long)points[i][0] * points[i][0]
                    + (long long)points[i][1] * points[i][1];
        pq.push({d, i});
        if ((int)pq.size() > k) pq.pop();
    }
    std::vector<std::vector<int>> out;
    while (!pq.empty()) { out.push_back(points[pq.top().second]); pq.pop(); }
    return out;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <utility>
#include <algorithm>
using std::vector;
static vector<vector<int>> norm(vector<vector<int>> v) { std::sort(v.begin(), v.end()); return v; }
//__USER__
int main() {
    { vector<vector<int>> p{{1,3},{-2,2}}; auto r = kClosest(p, 1);
      if (norm(r) != vector<vector<int>>({{-2,2}})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> p{{3,3},{5,-1},{-2,4}}; auto r = kClosest(p, 2);
      if (norm(r) != vector<vector<int>>({{-2,4},{3,3}})) { std::puts("case2"); return 1; } }
    { vector<vector<int>> p{{1,1}}; auto r = kClosest(p, 1);
      if (norm(r) != vector<vector<int>>({{1,1}})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> p{{0,1},{1,0}}; auto r = kClosest(p, 2);
      if (norm(r) != vector<vector<int>>({{0,1},{1,0}})) { std::puts("case4"); return 1; } }
    { vector<vector<int>> p{{2,2},{-1,0},{0,3}}; auto r = kClosest(p, 1);
      if (norm(r) != vector<vector<int>>({{-1,0}})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Rank points by squared distance to avoid floating point. A max-heap capped at size k holds the current k closest; whenever it overflows, evicting its farthest point maintains the invariant. After scanning everything the heap's contents are the answer. O(n log k) time, O(k) space; a quickselect on distances gives O(n) average time.
