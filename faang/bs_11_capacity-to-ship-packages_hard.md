## challenge: Capacity To Ship Packages Within D Days
tags: binary-search, array
track: faang
difficulty: hard

A conveyor belt has packages that must ship within `days` days. The `i`-th package has weight `weights[i]`. Each day the ship loads packages in the given order, never exceeding its capacity. Return the least ship capacity that lets all packages ship within `days` days.

Constraints: `1 <= days <= weights.length <= 5 * 10^4`, `1 <= weights[i] <= 500`.

Example: `weights = [1,2,3,4,5,6,7,8,9,10], days = 5` → `15`. Example: `days = 1` → `55`. Example: `days = 10` → `10`.

hint: Search over the capacity itself. A larger capacity never needs more days, so days-required is monotonic in capacity.
hint: For a candidate capacity, simulate: accumulate weights until the next one would overflow, then start a new day; count the days used.
hint: Capacity must be at least the heaviest single package (it has to fit) and at most the total weight (one day). Binary search that interval.

```cpp
// starter
#include <vector>
int shipWithinDays(std::vector<int>& weights, int days);
```

```cpp
int shipWithinDays(std::vector<int>& weights, int days) {
    long long lo = 0, hi = 0;
    for (int w : weights) { if (w > lo) lo = w; hi += w; }
    while (lo < hi) {
        long long mid = lo + (hi - lo) / 2;
        int used = 1;
        long long cur = 0;
        for (int w : weights) {
            if (cur + w > mid) { ++used; cur = w; }
            else cur += w;
        }
        if (used <= days) hi = mid;
        else lo = mid + 1;
    }
    return (int)lo;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<int> w{1,2,3,4,5,6,7,8,9,10}; if (shipWithinDays(w, 5)  != 15) { std::puts("case1"); return 1; } }
    { vector<int> w{1,2,3,4,5,6,7,8,9,10}; if (shipWithinDays(w, 1)  != 55) { std::puts("case2"); return 1; } }
    { vector<int> w{1,2,3,4,5,6,7,8,9,10}; if (shipWithinDays(w, 10) != 10) { std::puts("case3"); return 1; } }
    { vector<int> w{3,2,2,4,1,4};          if (shipWithinDays(w, 3)  != 6)  { std::puts("case4"); return 1; } }
    { vector<int> w{1,2,3,1,1};            if (shipWithinDays(w, 4)  != 3)  { std::puts("case5"); return 1; } }
    { vector<int> w{10};                   if (shipWithinDays(w, 1)  != 10) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Classic binary-search-on-answer. The predicate "capacity `C` ships everything within `days`" is monotonic: raising `C` can only reduce the days needed. Evaluate it in O(n) by greedily packing each day until the next package would overflow. Search `C` over `[max(weights), sum(weights)]` for the smallest feasible capacity. O(n log(sum)) time, O(1) space.
