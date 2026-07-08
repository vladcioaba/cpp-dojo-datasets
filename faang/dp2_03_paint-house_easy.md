## challenge: Paint House
tags: dynamic-programming, array
track: faang
difficulty: easy

A row of `n` houses must each be painted red, blue, or green. Painting house `i` a given color has a cost `costs[i][0]` (red), `costs[i][1]` (blue), or `costs[i][2]` (green). No two adjacent houses may share the same color. Return the minimum total cost to paint every house.

Constraints: `0 <= n <= 100`, `0 <= costs[i][j] <= 20`, each `costs[i]` has exactly three entries.

Example: `costs = [[17,2,17],[16,16,5],[14,3,19]]` → `10` (blue, green, blue: `2 + 5 + 3`). Example: `costs = [[7,6,2]]` → `2`.

hint: The cheapest way to paint house `i` a color adds that color's cost to the cheapest of the two other colors at house `i-1`.
hint: Carry just the three best running totals; you never need the full grid.

```cpp
// starter
#include <vector>
int minCost(std::vector<std::vector<int>>& costs);
```

```cpp
int minCost(std::vector<std::vector<int>>& costs) {
    if (costs.empty()) return 0;
    int r = costs[0][0], b = costs[0][1], g = costs[0][2];
    for (size_t i = 1; i < costs.size(); ++i) {
        int nr = costs[i][0] + std::min(b, g);
        int nb = costs[i][1] + std::min(r, g);
        int ng = costs[i][2] + std::min(r, b);
        r = nr; b = nb; g = ng;
    }
    return std::min({r, b, g});
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
    { vector<vector<int>> c{{17,2,17},{16,16,5},{14,3,19}}; if (minCost(c) != 10) { std::puts("case1"); return 1; } }
    { vector<vector<int>> c{{7,6,2}};                       if (minCost(c) != 2)  { std::puts("case2"); return 1; } }
    { vector<vector<int>> c{};                              if (minCost(c) != 0)  { std::puts("case3"); return 1; } }
    { vector<vector<int>> c{{1,2,3},{1,2,3},{1,2,3}};       if (minCost(c) != 4)  { std::puts("case4"); return 1; } }
    { vector<vector<int>> c{{5,8,6},{19,14,13},{7,5,12},{14,15,17},{3,20,10}}; if (minCost(c) != 43) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Let `red`, `blue`, `green` be the minimum cost to paint houses `0..i` with house `i` in each respective color. Because adjacent houses differ, house `i` painted red must sit on a house `i-1` that was blue or green, so `red_i = costs[i][0] + min(blue_{i-1}, green_{i-1})`, and symmetrically for the others. Three scalars updated per house give O(n) time and O(1) space.
