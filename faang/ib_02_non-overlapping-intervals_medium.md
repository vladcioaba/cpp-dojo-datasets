## challenge: Non-overlapping Intervals
tags: intervals, greedy, sorting
track: faang
difficulty: medium

Given an array of intervals, return the minimum number of intervals you must remove so that the remaining intervals are pairwise non-overlapping. Intervals that only touch at an endpoint (like `[1,2]` and `[2,3]`) are considered non-overlapping.

Constraints: `1 <= intervals.length <= 10^5`, `intervals[i].length == 2`, `-5*10^4 <= start < end <= 5*10^4`.

Example: `intervals = [[1,2],[2,3],[3,4],[1,3]]` → `1` (remove `[1,3]`). Example: `intervals = [[1,2],[1,2],[1,2]]` → `2`. Example: `intervals = [[1,2],[2,3]]` → `0`.

hint: This is the classic activity-selection problem in disguise: keeping the maximum number of non-overlapping intervals is equivalent to removing the fewest.
hint: Sort by end time. Greedily keep an interval whenever its start is at or after the end of the last kept interval; otherwise it must be removed.
hint: Track only the end of the most recently kept interval. If the current start is less than that end, increment the removal count; otherwise advance the kept end.

```cpp
// starter
#include <vector>
int eraseOverlapIntervals(std::vector<std::vector<int>>& intervals);
```

```cpp
int eraseOverlapIntervals(std::vector<std::vector<int>>& intervals) {
    if (intervals.empty()) return 0;
    std::sort(intervals.begin(), intervals.end(),
              [](const std::vector<int>& a, const std::vector<int>& b) { return a[1] < b[1]; });
    int end = intervals[0][1], removed = 0;
    for (int i = 1; i < (int)intervals.size(); ++i) {
        if (intervals[i][0] < end) ++removed;
        else end = intervals[i][1];
    }
    return removed;
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
    { vector<vector<int>> iv{{1,2},{2,3},{3,4},{1,3}};
      if (eraseOverlapIntervals(iv) != 1) { std::puts("case1"); return 1; } }
    { vector<vector<int>> iv{{1,2},{1,2},{1,2}};
      if (eraseOverlapIntervals(iv) != 2) { std::puts("case2"); return 1; } }
    { vector<vector<int>> iv{{1,2},{2,3}};
      if (eraseOverlapIntervals(iv) != 0) { std::puts("case3"); return 1; } }
    { vector<vector<int>> iv{{1,100},{11,22},{1,11},{2,12}};
      if (eraseOverlapIntervals(iv) != 2) { std::puts("case4"); return 1; } }
    { vector<vector<int>> iv{{-52,31}};
      if (eraseOverlapIntervals(iv) != 0) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Maximizing the count of intervals you keep minimizes the count you remove. Sorting by end time and greedily accepting the interval that finishes earliest leaves the most room for later intervals — the standard interval-scheduling argument. Whenever the next interval starts before the last accepted end, it overlaps and is counted as a removal; otherwise it becomes the new frontier. O(n log n) for the sort, O(1) extra space.
