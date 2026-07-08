## challenge: Maximum Number of Events That Can Be Attended
tags: heap, priority-queue, greedy, sorting

track: faang
difficulty: hard

You are given events as `events[i] = [startDay_i, endDay_i]`, meaning event `i` may be attended on any single day `d` with `startDay_i <= d <= endDay_i`. You can attend at most one event per day, and each event counts once. Return the maximum number of events you can attend.

Constraints: `1 <= events.length <= 10^5`, `1 <= startDay_i <= endDay_i <= 10^5`.

Example: `events = [[1,2],[2,3],[3,4]]` → `3` (attend one each on days 1, 2, 3). Example: `events = [[1,2],[2,3],[3,4],[1,2]]` → `4`. Example: `events = [[1,4],[4,4],[2,2],[3,4],[1,1]]` → `4`.

hint: Iterate day by day; on each day the events available are those whose window contains it.
hint: Among the currently available events, attend the one that ends soonest — it has the least remaining flexibility.
hint: Sort events by start day and use a min-heap of end days: each day add the newly-started events, drop the expired ones, then attend the earliest-ending available event.

```cpp
// starter
#include <vector>
int maxEvents(std::vector<std::vector<int>>& events);
```

```cpp
int maxEvents(std::vector<std::vector<int>>& events) {
    std::sort(events.begin(), events.end());
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;  // end days of available events
    int n = (int)events.size(), i = 0, count = 0, maxDay = 0;
    for (auto& e : events) maxDay = std::max(maxDay, e[1]);
    for (int day = 1; day <= maxDay; ++day) {
        while (i < n && events[i][0] == day) { pq.push(events[i][1]); ++i; }
        while (!pq.empty() && pq.top() < day) pq.pop();   // already expired
        if (!pq.empty()) { pq.pop(); ++count; }
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <functional>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> e{{1,2},{2,3},{3,4}};             if (maxEvents(e) != 3) { std::puts("case1"); return 1; } }
    { vector<vector<int>> e{{1,2},{2,3},{3,4},{1,2}};       if (maxEvents(e) != 4) { std::puts("case2"); return 1; } }
    { vector<vector<int>> e{{1,4},{4,4},{2,2},{3,4},{1,1}}; if (maxEvents(e) != 4) { std::puts("case3"); return 1; } }
    { vector<vector<int>> e{{1,100000}};                    if (maxEvents(e) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> e{{1,1},{1,1},{1,1}};             if (maxEvents(e) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sweep days in increasing order. Sort events by start so you can add each event to a min-heap of end days exactly when its window opens. Before choosing on a given day, discard any heap entries whose end day already passed, then attend the available event that ends soonest — the earliest-ending event is the greediest choice because postponing it risks losing it while later-ending events keep their options open. Each event is pushed and popped once, so the cost is O(n log n) plus the O(maxDay) day sweep, with O(n) space.
