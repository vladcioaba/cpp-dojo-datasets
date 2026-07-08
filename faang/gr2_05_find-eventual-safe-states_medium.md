## challenge: Find Eventual Safe States
tags: graph, dfs, cycle-detection, topological-sort
track: faang
difficulty: medium

There is a directed graph of `n` nodes labeled `0` to `n - 1`, given as `graph[i]`, the list of nodes reachable from node `i` by a single directed edge. A node is a terminal node if it has no outgoing edges. A node is a safe node if every possible path starting from it leads to a terminal node (or another safe node) in a finite number of steps — that is, no path from it can enter a cycle. Return an array containing all safe nodes in ascending order.

Constraints: `n == graph.length`, `1 <= n <= 10^4`, `0 <= graph[i].length <= n`, node values are distinct within each list, the graph may contain cycles.

Example: `graph = [[1,2],[2,3],[5],[0],[5],[],[]]` → `[2,4,5,6]`. Example: `graph = [[1,2,3,4],[1,2],[3,4],[0,4],[]]` → `[4]`.

hint: A node is unsafe exactly when it can reach a cycle. Color nodes to detect cycles during DFS.
hint: Use three states — unvisited, in-progress, safe. Encountering an in-progress node means a cycle; a node is safe only if all of its successors are safe.

```cpp
// starter
#include <vector>
std::vector<int> eventualSafeNodes(std::vector<std::vector<int>>& graph);
```

```cpp
#include <functional>
std::vector<int> eventualSafeNodes(std::vector<std::vector<int>>& graph) {
    int n = (int)graph.size();
    std::vector<int> color(n, 0); // 0 = unvisited, 1 = in-progress, 2 = safe
    std::function<bool(int)> dfs = [&](int u) -> bool {
        if (color[u] > 0) return color[u] == 2;
        color[u] = 1;
        for (int v : graph[u]) if (!dfs(v)) return false;
        color[u] = 2;
        return true;
    };
    std::vector<int> res;
    for (int i = 0; i < n; ++i) if (dfs(i)) res.push_back(i);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> g{{1,2},{2,3},{5},{0},{5},{},{}};
      auto r = eventualSafeNodes(g); vector<int> e{2,4,5,6};
      if (r != e) { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{1,2,3,4},{1,2},{3,4},{0,4},{}};
      auto r = eventualSafeNodes(g); vector<int> e{4};
      if (r != e) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{}};
      auto r = eventualSafeNodes(g); vector<int> e{0};
      if (r != e) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{1},{2},{0}};
      auto r = eventualSafeNodes(g); vector<int> e{};
      if (r != e) { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{1},{2},{}};
      auto r = eventualSafeNodes(g); vector<int> e{0,1,2};
      if (r != e) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A node is safe iff no path from it reaches a cycle. Run a DFS with three colors: unvisited, in-progress (on the current recursion stack), and safe. If DFS reaches an in-progress node, we have found a back edge and thus a cycle, so the node is unsafe. A node becomes safe only after all of its successors return safe. Collecting nodes by iterating `0..n-1` yields ascending order automatically. Each node and edge is examined once: O(n + E) time, O(n) space.
