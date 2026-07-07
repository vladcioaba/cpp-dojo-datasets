## challenge: Merge Intervals
tags: intervals, sorting, array
track: faang
difficulty: medium

Given an array of `intervals` where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals and return the non-overlapping intervals that cover all the input intervals. Intervals touching at an endpoint (e.g. `[1,4]` and `[4,5]`) overlap. Return them sorted by start.

Constraints: `1 <= intervals.length <= 10^4`, `intervals[i][0] <= intervals[i][1]`.

Example: `[[1,3],[2,6],[8,10],[15,18]]` → `[[1,6],[8,10],[15,18]]`. Example: `[[1,4],[4,5]]` → `[[1,5]]`.

hint: Once intervals are sorted by start, any overlap can only occur between consecutive intervals.
hint: Sort by start, then sweep, extending the last kept interval whenever the next one overlaps it.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> merge(std::vector<std::vector<int>>& intervals);
```

```cpp
std::vector<std::vector<int>> merge(std::vector<std::vector<int>>& intervals) {
    std::sort(intervals.begin(), intervals.end());
    std::vector<std::vector<int>> res;
    for (auto& iv : intervals) {
        if (!res.empty() && iv[0] <= res.back()[1])
            res.back()[1] = std::max(res.back()[1], iv[1]);
        else
            res.push_back(iv);
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
int main() {
    { vector<vector<int>> in{{1,3},{2,6},{8,10},{15,18}};
      if (merge(in) != vector<vector<int>>({{1,6},{8,10},{15,18}})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> in{{1,4},{4,5}};
      if (merge(in) != vector<vector<int>>({{1,5}})) { std::puts("case2"); return 1; } }
    { vector<vector<int>> in{{1,4},{0,4}};
      if (merge(in) != vector<vector<int>>({{0,4}})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> in{{1,4},{2,3}};
      if (merge(in) != vector<vector<int>>({{1,4}})) { std::puts("case4"); return 1; } }
    { vector<vector<int>> in{{5,6}};
      if (merge(in) != vector<vector<int>>({{5,6}})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort the intervals by start, then walk through them: if the current interval starts at or before the last merged interval's end, extend that end, otherwise append a new interval. Sorting dominates the cost. O(n log n) time, O(n) space (O(1) beyond the output).
