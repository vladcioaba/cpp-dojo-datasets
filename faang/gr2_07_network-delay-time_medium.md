## challenge: Network Delay Time
tags: graph, dijkstra, shortest-path
track: faang
difficulty: medium

You are given a network of `n` nodes labeled `1` to `n`, and a list `times` of travel times as directed edges `times[i] = [u, v, w]`, meaning a signal takes `w` time to go from node `u` to node `v`. A signal is sent from a given node `k`. Return the minimum time it takes for all `n` nodes to receive the signal. If it is impossible for all nodes to receive the signal, return `-1`.

Constraints: `1 <= k <= n <= 100`, `1 <= times.length <= 6000`, `1 <= u, v <= n`, `u != v`, `0 <= w <= 100`, all `(u, v)` pairs distinct.

Example: `times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2` → `2`. Example: `times = [[1,2,1]], n = 2, k = 2` → `-1` (node 1 is never reached).

hint: This is a single-source shortest path from `k`. The answer is the largest shortest-path distance.
hint: Run Dijkstra with a min-heap from `k`. If any node stays at infinity it cannot be reached, so return `-1`; otherwise return the maximum distance.

```cpp
// starter
#include <vector>
int networkDelayTime(std::vector<std::vector<int>>& times, int n, int k);
```

```cpp
#include <queue>
#include <utility>
#include <climits>
#include <algorithm>
int networkDelayTime(std::vector<std::vector<int>>& times, int n, int k) {
    std::vector<std::vector<std::pair<int, int>>> g(n + 1);
    for (auto& t : times) g[t[0]].push_back({t[1], t[2]});
    std::vector<int> dist(n + 1, INT_MAX);
    dist[k] = 0;
    std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>,
                        std::greater<std::pair<int, int>>> pq;
    pq.push({0, k});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;
        for (auto& [v, w] : g[u]) {
            if (d + w < dist[v]) { dist[v] = d + w; pq.push({dist[v], v}); }
        }
    }
    int ans = 0;
    for (int i = 1; i <= n; ++i) {
        if (dist[i] == INT_MAX) return -1;
        ans = std::max(ans, dist[i]);
    }
    return ans;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> t{{2,1,1},{2,3,1},{3,4,1}}; if (networkDelayTime(t, 4, 2) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> t{{1,2,1}}; if (networkDelayTime(t, 2, 1) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> t{{1,2,1}}; if (networkDelayTime(t, 2, 2) != -1) { std::puts("case3"); return 1; } }
    { vector<vector<int>> t{{1,2,1},{2,3,2},{1,3,4}}; if (networkDelayTime(t, 3, 1) != 3) { std::puts("case4"); return 1; } }
    { vector<vector<int>> t{}; if (networkDelayTime(t, 1, 1) != 0) { std::puts("case5"); return 1; } }
    { vector<vector<int>> t{{1,2,1},{1,3,1},{3,4,5}}; if (networkDelayTime(t, 4, 1) != 6) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The time for all nodes to receive the signal is the maximum over the shortest-path distances from `k` to every node. Compute these with Dijkstra using a min-heap keyed by tentative distance, relaxing outgoing edges. After the run, if any node is still at infinity it is unreachable and the answer is `-1`; otherwise return the maximum finite distance. With E edges and V nodes the cost is O(E log V) time and O(V + E) space.
