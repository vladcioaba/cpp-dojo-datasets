## challenge: Car Fleet
tags: monotonic-stack, stack, sorting
track: faang
difficulty: hard

There are `n` cars on a one-lane road heading to a destination `target` miles away. Car `i` starts at `position[i]` (all distinct) and drives at constant `speed[i]`. A faster car cannot pass a slower one ahead of it; instead it catches up and they travel together as a single "car fleet" at the slower car's speed. A car that catches the fleet exactly at the destination still counts as joining it. Return the number of distinct car fleets that arrive at the destination.

Constraints: `1 <= n <= 10^5`; `0 < target <= 10^6`; `0 <= position[i] < target`; `0 < speed[i] <= 10^6`; all `position[i]` are distinct.

Example: `target = 12, position = [10,8,0,5,3], speed = [2,4,1,1,3]` → `3`. Example: `target = 10, position = [3], speed = [3]` → `1`. Example: `target = 100, position = [0,2,4], speed = [4,2,1]` → `1`.

hint: Sort the cars by starting position from closest-to-target to farthest; a car can only ever catch a car ahead of it.
hint: The relevant quantity per car is its time to reach the target: `(target - position) / speed`.
hint: Process cars front to back keeping the lead fleet's arrival time; a car whose time exceeds the lead is slower, so it starts a new fleet — otherwise it merges into the lead.

```cpp
// starter
#include <vector>
int carFleet(int target, std::vector<int>& position, std::vector<int>& speed);
```

```cpp
int carFleet(int target, std::vector<int>& position, std::vector<int>& speed) {
    int n = (int)position.size();
    std::vector<int> idx(n);
    for (int i = 0; i < n; ++i) idx[i] = i;
    std::sort(idx.begin(), idx.end(),
              [&](int a, int b) { return position[a] > position[b]; });
    int fleets = 0;
    double lead = 0.0;   // arrival time of the current front fleet
    for (int i = 0; i < n; ++i) {
        double t = double(target - position[idx[i]]) / speed[idx[i]];
        if (t > lead) { ++fleets; lead = t; }  // slower than the fleet ahead: new fleet
    }
    return fleets;
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
    { vector<int> p{10,8,0,5,3}, s{2,4,1,1,3}; if (carFleet(12, p, s) != 3) { std::puts("case1"); return 1; } }
    { vector<int> p{3}, s{3};                   if (carFleet(10, p, s) != 1) { std::puts("case2"); return 1; } }
    { vector<int> p{0,2,4}, s{4,2,1};           if (carFleet(100, p, s) != 1) { std::puts("case3"); return 1; } }
    { vector<int> p{6,8}, s{3,2};               if (carFleet(10, p, s) != 2) { std::puts("case4"); return 1; } }
    { vector<int> p{0,4,2}, s{2,1,3};           if (carFleet(10, p, s) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Sort cars by position, closest to the target first, since a car can only be blocked by one ahead of it. For each car compute its unobstructed arrival time `(target - position) / speed`. Sweep from the front maintaining the arrival time of the current lead fleet: if a car's time is greater than the lead's, it is slower and cannot catch up, so it forms a new fleet and becomes the new lead; otherwise it catches the fleet ahead (by the target at latest) and merges. Sorting dominates at O(n log n) time with O(n) space.
