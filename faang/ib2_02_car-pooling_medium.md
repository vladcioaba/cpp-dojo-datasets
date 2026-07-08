## challenge: Car Pooling
tags: intervals, prefix-sum, sorting
track: faang
difficulty: medium

A car with `capacity` empty seats drives east along a one-directional road. You are given `trips` where `trips[i] = [numPassengers_i, from_i, to_i]` means a group of `numPassengers_i` boards at location `from_i` and leaves at location `to_i`. The locations are distances east of the start. Return `true` if it is possible to pick up and drop off every group without ever exceeding `capacity`, otherwise `false`. Passengers who leave at a location free their seats before passengers boarding at that same location.

Constraints: `1 <= trips.length <= 1000`, `1 <= numPassengers_i <= 100`, `0 <= from_i < to_i <= 1000`, `1 <= capacity <= 10^5`.

Example: `trips = [[2,1,5],[3,3,7]], capacity = 4` → `false`. Example: `trips = [[2,1,5],[3,3,7]], capacity = 5` → `true`.

hint: Think of each trip as an interval `[from, to)` that adds passengers; you want the maximum concurrent load.
hint: A difference array over the 1001 possible locations records `+p` at `from` and `-p` at `to`; a running prefix sum reconstructs the load at each point.
hint: If the running load ever exceeds `capacity`, the answer is `false`. Dropping off happens at `to` before boarding, which the `-p` at `to` handles naturally.

```cpp
// starter
#include <vector>
bool carPooling(std::vector<std::vector<int>>& trips, int capacity);
```

```cpp
bool carPooling(std::vector<std::vector<int>>& trips, int capacity) {
    int diff[1001] = {0};
    for (auto& t : trips) {
        diff[t[1]] += t[0];
        diff[t[2]] -= t[0];
    }
    int load = 0;
    for (int i = 0; i <= 1000; ++i) {
        load += diff[i];
        if (load > capacity) return false;
    }
    return true;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> t{{2,1,5},{3,3,7}};
      if (carPooling(t, 4) != false) { std::puts("case1"); return 1; } }
    { vector<vector<int>> t{{2,1,5},{3,3,7}};
      if (carPooling(t, 5) != true) { std::puts("case2"); return 1; } }
    { vector<vector<int>> t{{2,1,5},{3,5,7}};
      if (carPooling(t, 3) != true) { std::puts("case3"); return 1; } }
    { vector<vector<int>> t{{3,2,7},{3,7,9},{8,3,9}};
      if (carPooling(t, 11) != true) { std::puts("case4"); return 1; } }
    { vector<vector<int>> t{{9,0,1},{3,3,7}};
      if (carPooling(t, 4) != false) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Each trip is an interval that raises the passenger count on `[from, to)`. A difference array records `+p` at `from` and `-p` at `to`; sweeping a prefix sum from location 0 upward reconstructs the exact load everywhere. Because a drop-off at a location is applied (via the `-p` at `to`) at the same index where later boardings are added, seats free up in time. If any prefix sum exceeds `capacity` the schedule is infeasible. With locations bounded by 1000 this is O(n + maxLoc) time and O(maxLoc) space.
