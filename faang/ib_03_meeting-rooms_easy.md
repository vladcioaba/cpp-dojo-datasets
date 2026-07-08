## challenge: Meeting Rooms
tags: intervals, sorting, array
track: faang
difficulty: easy

Given an array of meeting time intervals where `intervals[i] = [start_i, end_i]`, determine whether a single person could attend all meetings. Two meetings conflict if they overlap; meetings that merely touch at an endpoint (one ends exactly when the next begins) do not conflict.

Constraints: `0 <= intervals.length <= 10^4`, `intervals[i].length == 2`, `0 <= start_i < end_i <= 10^6`.

Example: `intervals = [[0,30],[5,10],[15,20]]` → `false` (the `[0,30]` meeting overlaps the others). Example: `intervals = [[7,10],[2,4]]` → `true`.

hint: Sort the meetings by start time so that conflicts can only occur between neighbors.
hint: After sorting, a conflict exists exactly when some meeting starts before the previous one ends.
hint: Compare `intervals[i][0]` against `intervals[i-1][1]`; a strict `<` means overlap. Touching endpoints (`==`) are allowed.

```cpp
// starter
#include <vector>
bool canAttendMeetings(std::vector<std::vector<int>>& intervals);
```

```cpp
bool canAttendMeetings(std::vector<std::vector<int>>& intervals) {
    std::sort(intervals.begin(), intervals.end());
    for (int i = 1; i < (int)intervals.size(); ++i)
        if (intervals[i][0] < intervals[i - 1][1]) return false;
    return true;
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
    { vector<vector<int>> iv{{0,30},{5,10},{15,20}};
      if (canAttendMeetings(iv) != false) { std::puts("case1"); return 1; } }
    { vector<vector<int>> iv{{7,10},{2,4}};
      if (canAttendMeetings(iv) != true) { std::puts("case2"); return 1; } }
    { vector<vector<int>> iv;
      if (canAttendMeetings(iv) != true) { std::puts("case3"); return 1; } }
    { vector<vector<int>> iv{{1,5},{5,8},{8,9}};
      if (canAttendMeetings(iv) != true) { std::puts("case4"); return 1; } }
    { vector<vector<int>> iv{{5,8},{1,6}};
      if (canAttendMeetings(iv) != false) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort by start time; after sorting, the only possible overlap for meeting `i` is with meeting `i-1`. If any meeting begins strictly before its predecessor ends, the person cannot attend both. Endpoints that touch (`prevEnd == curStart`) are fine because one meeting is over the instant the next begins. O(n log n) time dominated by the sort, O(1) extra space.
