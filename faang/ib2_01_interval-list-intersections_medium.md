## challenge: Interval List Intersections
tags: intervals, two-pointers, array
track: faang
difficulty: medium

You are given two lists of closed intervals, `firstList` and `secondList`, where each list is pairwise disjoint and sorted in ascending order by start. Return the intersection of these two interval lists. A closed interval `[a, b]` (with `a <= b`) denotes the set of real numbers `x` with `a <= x <= b`; the intersection of two closed intervals is either empty or another closed interval. Return the intersections in sorted order.

Constraints: `0 <= firstList.length, secondList.length <= 1000`, `firstList.length + secondList.length >= 1`, `0 <= start_i < end_i <= 10^9`, each list is disjoint and sorted by start.

Example: `firstList = [[0,2],[5,10],[13,23],[24,25]], secondList = [[1,5],[8,12],[15,24],[25,26]]` → `[[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]]`. Example: `firstList = [[1,3],[5,9]], secondList = []` → `[]`.

hint: Walk both lists with two pointers; the overlap of the two current intervals, if any, is `[max(starts), min(ends)]`.
hint: An overlap exists exactly when `max(a0, b0) <= min(a1, b1)`; only then push it to the result.
hint: After comparing, advance the pointer whose interval ends first — its interval can never intersect anything later.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> intervalIntersection(std::vector<std::vector<int>>& firstList, std::vector<std::vector<int>>& secondList);
```

```cpp
std::vector<std::vector<int>> intervalIntersection(std::vector<std::vector<int>>& firstList, std::vector<std::vector<int>>& secondList) {
    std::vector<std::vector<int>> res;
    int i = 0, j = 0;
    int n = (int)firstList.size(), m = (int)secondList.size();
    while (i < n && j < m) {
        int lo = std::max(firstList[i][0], secondList[j][0]);
        int hi = std::min(firstList[i][1], secondList[j][1]);
        if (lo <= hi) res.push_back({lo, hi});
        if (firstList[i][1] < secondList[j][1]) ++i;
        else ++j;
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
//__USER__
static bool eq(const vector<vector<int>>& a, const vector<vector<int>>& b) { return a == b; }
int main() {
    { vector<vector<int>> a{{0,2},{5,10},{13,23},{24,25}}, b{{1,5},{8,12},{15,24},{25,26}};
      auto r = intervalIntersection(a, b);
      if (!eq(r, {{1,2},{5,5},{8,10},{15,23},{24,24},{25,25}})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> a{{1,3},{5,9}}, b;
      auto r = intervalIntersection(a, b);
      if (!eq(r, {})) { std::puts("case2"); return 1; } }
    { vector<vector<int>> a, b{{4,8},{10,12}};
      auto r = intervalIntersection(a, b);
      if (!eq(r, {})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> a{{1,7}}, b{{3,10}};
      auto r = intervalIntersection(a, b);
      if (!eq(r, {{3,7}})) { std::puts("case4"); return 1; } }
    { vector<vector<int>> a{{1,5}}, b{{5,9}};
      auto r = intervalIntersection(a, b);
      if (!eq(r, {{5,5}})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because both lists are sorted and internally disjoint, a two-pointer sweep suffices. At each step the only candidate overlap is between the two current intervals: `[max(starts), min(ends)]`, which is valid iff `lo <= hi`. Whichever interval ends earlier cannot intersect any later interval of the other list, so we advance that pointer. Each pointer moves forward at most `n + m` times, giving O(n + m) time and O(1) auxiliary space beyond the output.
