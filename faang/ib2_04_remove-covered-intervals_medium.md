## challenge: Remove Covered Intervals
tags: intervals, sorting, greedy
track: faang
difficulty: medium

Given an array `intervals` where `intervals[i] = [l_i, r_i]` represents the interval `[l_i, r_i)`, remove every interval that is covered by another interval in the list. The interval `[a, b)` is covered by `[c, d)` if and only if `c <= a` and `b <= d`. Return the number of remaining intervals after removing all covered ones.

Constraints: `1 <= intervals.length <= 1000`, `intervals[i].length == 2`, `0 <= l_i < r_i <= 10^5`, all the given intervals are unique.

Example: `intervals = [[1,4],[3,6],[2,8]]` → `2` (only `[3,6]` is covered, by `[2,8]`). Example: `intervals = [[1,4],[2,3]]` → `1`.

hint: Sort so that a covering interval always appears before the intervals it might cover.
hint: Sort by start ascending; break ties by end descending so a wider interval precedes a narrower one sharing the same start.
hint: Sweep left to right tracking the largest end seen; an interval is covered exactly when its end does not exceed that running maximum.

```cpp
// starter
#include <vector>
int removeCoveredIntervals(std::vector<std::vector<int>>& intervals);
```

```cpp
int removeCoveredIntervals(std::vector<std::vector<int>>& intervals) {
    std::sort(intervals.begin(), intervals.end(), [](const std::vector<int>& a, const std::vector<int>& b) {
        if (a[0] != b[0]) return a[0] < b[0];
        return a[1] > b[1];
    });
    int count = 0, prevEnd = -1;
    for (auto& iv : intervals) {
        if (iv[1] > prevEnd) { ++count; prevEnd = iv[1]; }
    }
    return count;
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
    { vector<vector<int>> iv{{1,4},{3,6},{2,8}};
      if (removeCoveredIntervals(iv) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> iv{{1,4},{2,3}};
      if (removeCoveredIntervals(iv) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> iv{{0,10},{5,12}};
      if (removeCoveredIntervals(iv) != 2) { std::puts("case3"); return 1; } }
    { vector<vector<int>> iv{{3,10},{4,10},{5,11}};
      if (removeCoveredIntervals(iv) != 2) { std::puts("case4"); return 1; } }
    { vector<vector<int>> iv{{1,2},{1,4},{3,4}};
      if (removeCoveredIntervals(iv) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort by start ascending, breaking ties by end descending. After sorting, any covering interval appears no later than the intervals it covers, and among equal starts the widest comes first. Sweep once, tracking the maximum end seen so far: an interval survives only if its end strictly exceeds that maximum (otherwise its `[l, r)` lies inside a previously counted interval, since its start is already `>=`). Counting survivors yields the answer in O(n log n) time and O(1) extra space.
