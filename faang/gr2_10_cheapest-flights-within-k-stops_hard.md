## challenge: Cheapest Flights Within K Stops
tags: graph, bellman-ford, shortest-path, dynamic-programming
track: faang
difficulty: hard

There are `n` cities labeled `0` to `n - 1` connected by some flights. You are given `flights` where `flights[i] = [from, to, price]` denotes a flight from city `from` to city `to` with cost `price`. You are also given three integers `src`, `dst`, and `k`. Return the cheapest price to travel from `src` to `dst` using at most `k` stops (that is, at most `k + 1` edges). If there is no such route, return `-1`.

Constraints: `1 <= n <= 100`, `0 <= flights.length <= n * (n - 1)`, `flights[i].length == 3`, `0 <= from, to < n`, `from != to`, `1 <= price <= 10^4`, `0 <= src, dst < n`, `src != dst`, `0 <= k < n`.

Example: `n = 4, flights = [[0,1,100],[1,2,100],[2,0,100],[1,3,600],[2,3,200]], src = 0, dst = 3, k = 1` → `700` (0→1→3). Example: `n = 3, flights = [[0,1,100],[1,2,100],[0,2,500]], src = 0, dst = 2, k = 0` → `500`.

hint: The stop limit makes plain Dijkstra awkward. Think about relaxing edges a bounded number of times.
hint: Run Bellman-Ford for `k + 1` rounds. Each round relaxes edges from a frozen snapshot of the previous round so a path never uses more than the allowed number of edges.

```cpp
// starter
#include <vector>
int findCheapestPrice(int n, std::vector<std::vector<int>>& flights, int src, int dst, int k);
```

```cpp
#include <climits>
int findCheapestPrice(int n, std::vector<std::vector<int>>& flights, int src, int dst, int k) {
    std::vector<int> dist(n, INT_MAX);
    dist[src] = 0;
    for (int i = 0; i <= k; ++i) {
        std::vector<int> tmp = dist;
        for (auto& f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (dist[u] != INT_MAX && dist[u] + w < tmp[v]) tmp[v] = dist[u] + w;
        }
        dist = tmp;
    }
    return dist[dst] == INT_MAX ? -1 : dist[dst];
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> f{{0,1,100},{1,2,100},{2,0,100},{1,3,600},{2,3,200}};
      if (findCheapestPrice(4, f, 0, 3, 1) != 700) { std::puts("case1"); return 1; } }
    { vector<vector<int>> f{{0,1,100},{1,2,100},{0,2,500}};
      if (findCheapestPrice(3, f, 0, 2, 1) != 200) { std::puts("case2"); return 1; } }
    { vector<vector<int>> f{{0,1,100},{1,2,100},{0,2,500}};
      if (findCheapestPrice(3, f, 0, 2, 0) != 500) { std::puts("case3"); return 1; } }
    { vector<vector<int>> f{{0,1,2},{1,2,1},{2,0,10}};
      if (findCheapestPrice(3, f, 1, 0, 1) != 11) { std::puts("case4"); return 1; } }
    { vector<vector<int>> f{{0,1,5}};
      if (findCheapestPrice(2, f, 1, 0, 1) != -1) { std::puts("case5"); return 1; } }
    { vector<vector<int>> f{{0,1,1},{1,2,1},{2,3,1},{0,3,10}};
      if (findCheapestPrice(4, f, 0, 3, 1) != 10) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The at-most-`k`-stops constraint bounds the number of edges to `k + 1`, which is exactly what Bellman-Ford's round structure captures. Initialize `dist[src] = 0`. For each of `k + 1` rounds, relax every edge but read distances from a frozen copy `tmp` of the previous round; freezing prevents a single round from chaining multiple edges, so after round `r` the distances reflect paths of at most `r` edges. The answer is `dist[dst]`, or `-1` if it never becomes finite. Time is O(k * E), space O(n).
