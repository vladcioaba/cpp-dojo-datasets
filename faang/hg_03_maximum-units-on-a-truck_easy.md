## challenge: Maximum Units on a Truck
tags: greedy, sorting, array
track: faang
difficulty: easy

You are given `boxTypes` where `boxTypes[i] = [numberOfBoxes, unitsPerBox]`, and an integer `truckSize` (the maximum total number of boxes the truck can carry). Choose boxes to maximize the total number of units carried; return that maximum.

Constraints: `1 <= boxTypes.length <= 1000`, `1 <= numberOfBoxes, unitsPerBox <= 1000`, `1 <= truckSize <= 10^6`.

Example: `boxTypes = [[1,3],[2,2],[3,1]], truckSize = 4` → `8` (take 1 box of 3, 2 boxes of 2, 1 box of 1 = 3+4+1). Example: `boxTypes = [[5,10],[2,5],[4,7],[3,9]], truckSize = 10` → `91`.

hint: Every box occupies exactly one unit of capacity, so you want the densest boxes first.
hint: Sort box types by `unitsPerBox` descending and fill the truck greedily.
hint: From each type take as many boxes as fit (`min(count, remaining)`) until the truck is full.

```cpp
// starter
#include <vector>
int maximumUnits(std::vector<std::vector<int>>& boxTypes, int truckSize);
```

```cpp
int maximumUnits(std::vector<std::vector<int>>& boxTypes, int truckSize) {
    std::sort(boxTypes.begin(), boxTypes.end(),
              [](const std::vector<int>& a, const std::vector<int>& b){ return a[1] > b[1]; });
    int units = 0;
    for (auto& b : boxTypes) {
        int take = std::min(b[0], truckSize);
        units += take * b[1];
        truckSize -= take;
        if (truckSize == 0) break;
    }
    return units;
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
    { vector<vector<int>> b{{1,3},{2,2},{3,1}};        if (maximumUnits(b, 4) != 8)  { std::puts("case1"); return 1; } }
    { vector<vector<int>> b{{5,10},{2,5},{4,7},{3,9}}; if (maximumUnits(b, 10) != 91){ std::puts("case2"); return 1; } }
    { vector<vector<int>> b{{1,1}};                    if (maximumUnits(b, 1) != 1)  { std::puts("case3"); return 1; } }
    { vector<vector<int>> b{{10,2}};                   if (maximumUnits(b, 3) != 6)  { std::puts("case4"); return 1; } }
    { vector<vector<int>> b{{2,3},{2,3}};              if (maximumUnits(b, 10) != 12){ std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Capacity is measured in boxes, and each box of a type contributes the same number of units, so the optimal exchange-argument choice is to load the highest `unitsPerBox` types first. Sort descending by units per box and greedily take `min(available, remaining)` boxes from each. O(n log n) time, O(1) extra space.
