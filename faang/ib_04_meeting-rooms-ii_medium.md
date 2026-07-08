## challenge: Meeting Rooms II
tags: intervals, sorting, greedy
track: faang
difficulty: medium

Given an array of meeting time intervals `intervals[i] = [start_i, end_i]`, return the minimum number of conference rooms required so that no two overlapping meetings share a room. A meeting that starts exactly when another ends may reuse the freed room.

Constraints: `0 <= intervals.length <= 10^4`, `intervals[i].length == 2`, `0 <= start_i < end_i <= 10^6`.

Example: `intervals = [[0,30],[5,10],[15,20]]` → `2`. Example: `intervals = [[7,10],[2,4]]` → `1`.

hint: The answer is the maximum number of meetings that are simultaneously in progress at any instant.
hint: Separate all start times and all end times into two sorted arrays; then sweep a timeline merging the two.
hint: Walk the start times; before opening a room for a start, release every room whose end time is `<= ` that start. The peak number of concurrently open rooms is the answer.

```cpp
// starter
#include <vector>
int minMeetingRooms(std::vector<std::vector<int>>& intervals);
```

```cpp
int minMeetingRooms(std::vector<std::vector<int>>& intervals) {
    int n = (int)intervals.size();
    std::vector<int> starts(n), ends(n);
    for (int i = 0; i < n; ++i) { starts[i] = intervals[i][0]; ends[i] = intervals[i][1]; }
    std::sort(starts.begin(), starts.end());
    std::sort(ends.begin(), ends.end());
    int rooms = 0, best = 0, j = 0;
    for (int i = 0; i < n; ++i) {
        while (j < n && ends[j] <= starts[i]) { --rooms; ++j; }
        ++rooms;
        best = std::max(best, rooms);
    }
    return best;
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
      if (minMeetingRooms(iv) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> iv{{7,10},{2,4}};
      if (minMeetingRooms(iv) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> iv;
      if (minMeetingRooms(iv) != 0) { std::puts("case3"); return 1; } }
    { vector<vector<int>> iv{{1,5},{8,9},{8,9}};
      if (minMeetingRooms(iv) != 2) { std::puts("case4"); return 1; } }
    { vector<vector<int>> iv{{1,10},{2,7},{3,19},{8,12},{10,20},{11,30}};
      if (minMeetingRooms(iv) != 4) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The minimum number of rooms equals the maximum overlap depth. Splitting starts and ends into two sorted arrays lets us sweep chronologically: for each start we first free every room whose meeting has already ended (`end <= start`, so a touching room is reused), then occupy a room. The running count of occupied rooms peaks at the answer. O(n log n) time, O(n) space.
