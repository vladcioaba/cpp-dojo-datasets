## challenge: Minimum Number of Refueling Stops
tags: heap, priority-queue, greedy, dynamic-programming
track: faang
difficulty: hard

A car starts with `startFuel` litres and must reach a destination `target` miles away; one litre drives one mile. Along the way `stations[i] = [position, fuel]` gives a station's distance from the start and the litres it can add. Return the minimum number of refuelling stops to reach the target, or `-1` if it cannot be reached. You may pass a station and refuel there later.

Constraints: `1 <= target, startFuel <= 10^9`, `0 <= stations.length <= 500`, stations are sorted by strictly increasing `position`, `1 <= position < target`, `1 <= fuel <= 10^9`.

Example: `target = 100, startFuel = 10, stations = [[10,60],[20,30],[30,30],[60,40]]` → `2`. Example: `target = 100, startFuel = 1, stations = [[10,100]]` → `-1`.

hint: Refuelling anywhere you have already driven past is allowed, so defer the decision of *which* station to use.
hint: Whenever you run short of fuel, the best station to have used is the passed one offering the most fuel.
hint: A max-heap of the fuel amounts of all reachable-but-unused stations lets you top up greedily, counting one stop per pop.

```cpp
// starter
#include <vector>
int minRefuelStops(int target, int startFuel, std::vector<std::vector<int>>& stations);
```

```cpp
int minRefuelStops(int target, int startFuel, std::vector<std::vector<int>>& stations) {
    std::priority_queue<int> pq;
    int n = (int)stations.size();
    int i = 0, stops = 0;
    long long fuel = startFuel;
    while (fuel < target) {
        while (i < n && stations[i][0] <= fuel) { pq.push(stations[i][1]); ++i; }
        if (pq.empty()) return -1;
        fuel += pq.top(); pq.pop();
        ++stops;
    }
    return stops;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> s{{10,60},{20,30},{30,30},{60,40}};
      if (minRefuelStops(100, 10, s) != 2)  { std::puts("case1"); return 1; } }
    { vector<vector<int>> s{{10,100}};
      if (minRefuelStops(100, 1, s) != -1)  { std::puts("case2"); return 1; } }
    { vector<vector<int>> s{};
      if (minRefuelStops(1, 1, s) != 0)     { std::puts("case3"); return 1; } }
    { vector<vector<int>> s{{25,25},{50,50}};
      if (minRefuelStops(100, 50, s) != 1)  { std::puts("case4"); return 1; } }
    { vector<vector<int>> s{{10,60},{20,30},{30,30},{60,40}};
      if (minRefuelStops(60, 10, s) != 1)   { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Because fuel from any already-passed station can be claimed at any time, drive as far as the current fuel allows while pushing every reachable station's fuel into a max-heap. When you cannot advance to the target, pop the largest deferred fuel and count a stop; if the heap is empty you are stranded. Each station is pushed and popped at most once. O(n log n) time, O(n) space.
