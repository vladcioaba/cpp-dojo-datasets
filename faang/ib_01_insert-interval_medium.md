## challenge: Insert Interval
tags: intervals, array, sorting
track: faang
difficulty: medium

You are given a list of non-overlapping intervals `intervals` sorted in ascending order by start, and a `newInterval`. Insert `newInterval` into the list so that the result is still sorted and has no overlapping intervals (merging where necessary). Return the resulting list of intervals.

Constraints: `0 <= intervals.length <= 10^4`, `intervals[i].length == 2`, `0 <= start <= end <= 10^5`, the input list is sorted by start and non-overlapping.

Example: `intervals = [[1,3],[6,9]], newInterval = [2,5]` → `[[1,5],[6,9]]`. Example: `intervals = [[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval = [4,8]` → `[[1,2],[3,10],[12,16]]`.

hint: The intervals are already sorted, so you can walk them once in three phases instead of re-sorting after insertion.
hint: Phase one copies every interval that ends before the new interval starts. Phase two absorbs every interval that overlaps the new one by expanding its bounds. Phase three copies the rest.
hint: An interval `intervals[i]` overlaps the (growing) new interval when `intervals[i][0] <= hi`; while that holds, take `lo = min(lo, start)` and `hi = max(hi, end)`.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> insert(std::vector<std::vector<int>>& intervals, std::vector<int>& newInterval);
```

```cpp
std::vector<std::vector<int>> insert(std::vector<std::vector<int>>& intervals, std::vector<int>& newInterval) {
    std::vector<std::vector<int>> res;
    int i = 0, n = (int)intervals.size();
    while (i < n && intervals[i][1] < newInterval[0]) res.push_back(intervals[i++]);
    int lo = newInterval[0], hi = newInterval[1];
    while (i < n && intervals[i][0] <= hi) {
        lo = std::min(lo, intervals[i][0]);
        hi = std::max(hi, intervals[i][1]);
        ++i;
    }
    res.push_back({lo, hi});
    while (i < n) res.push_back(intervals[i++]);
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
    { vector<vector<int>> iv{{1,3},{6,9}}; vector<int> ni{2,5};
      auto r = insert(iv, ni);
      if (!eq(r, {{1,5},{6,9}})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> iv{{1,2},{3,5},{6,7},{8,10},{12,16}}; vector<int> ni{4,8};
      auto r = insert(iv, ni);
      if (!eq(r, {{1,2},{3,10},{12,16}})) { std::puts("case2"); return 1; } }
    { vector<vector<int>> iv; vector<int> ni{5,7};
      auto r = insert(iv, ni);
      if (!eq(r, {{5,7}})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> iv{{1,5}}; vector<int> ni{2,3};
      auto r = insert(iv, ni);
      if (!eq(r, {{1,5}})) { std::puts("case4"); return 1; } }
    { vector<vector<int>> iv{{1,2},{5,6}}; vector<int> ni{0,9};
      auto r = insert(iv, ni);
      if (!eq(r, {{0,9}})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because the input is already sorted and disjoint, a single pass in three phases suffices. First push every interval strictly to the left of the new one (`end < newStart`). Then, while the next interval starts at or before the running upper bound `hi`, merge it by widening `[lo, hi]`; push the merged interval once. Finally push the untouched tail. O(n) time, O(n) output space.
