## challenge: Gas Station
tags: greedy, array
track: faang
difficulty: medium

There are `n` gas stations in a circle. `gas[i]` is the fuel available at station `i` and `cost[i]` is the fuel needed to drive from station `i` to `i + 1`. Starting with an empty tank at some station, return the index of the station from which you can complete the whole circuit once, or `-1` if impossible. The answer is unique when it exists.

Constraints: `n == gas.length == cost.length`, `1 <= n <= 10^5`, `0 <= gas[i], cost[i] <= 10^4`.

Example: `gas = [1,2,3,4,5], cost = [3,4,5,1,2]` → `3`. Example: `gas = [2,3,4], cost = [3,4,3]` → `-1`.

hint: A full circuit is possible only if total gas is at least total cost.
hint: If the running tank ever goes negative, none of the stations from the current start up to here can be the answer.
hint: When the tank drops below zero, reset the candidate start to the next station and clear the tank.

```cpp
// starter
#include <vector>
int canCompleteCircuit(std::vector<int>& gas, std::vector<int>& cost);
```

```cpp
int canCompleteCircuit(std::vector<int>& gas, std::vector<int>& cost) {
    int total = 0, tank = 0, start = 0;
    for (int i = 0; i < (int)gas.size(); ++i) {
        int diff = gas[i] - cost[i];
        total += diff;
        tank += diff;
        if (tank < 0) { start = i + 1; tank = 0; }
    }
    return total >= 0 ? start : -1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> g{1,2,3,4,5}, c{3,4,5,1,2}; if (canCompleteCircuit(g, c) != 3)  { std::puts("case1"); return 1; } }
    { vector<int> g{2,3,4}, c{3,4,3};         if (canCompleteCircuit(g, c) != -1) { std::puts("case2"); return 1; } }
    { vector<int> g{5}, c{4};                 if (canCompleteCircuit(g, c) != 0)  { std::puts("case3"); return 1; } }
    { vector<int> g{3,1,1}, c{1,2,2};         if (canCompleteCircuit(g, c) != 0)  { std::puts("case4"); return 1; } }
    { vector<int> g{2,3,4}, c{3,4,4};         if (canCompleteCircuit(g, c) != -1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** If the summed surplus `gas[i] - cost[i]` is negative no start works; otherwise a start always exists and is unique. Track a running tank from a candidate start: the moment it dips below zero, every station in that window fails, so jump the candidate to the next index and reset the tank. Because total surplus is non-negative, the final candidate completes the loop. O(n) time, O(1) space.
