## challenge: All Paths From Source to Target
tags: graph, dfs, backtracking
track: faang
difficulty: medium

Given a directed acyclic graph (DAG) of `n` nodes labeled `0` to `n - 1`, find all possible paths from node `0` to node `n - 1` and return them in any order. The graph is given as `graph[i]`, a list of all nodes you can visit from node `i` via a directed edge. Each returned path lists the node labels in the order visited.

Constraints: `n == graph.length`, `2 <= n <= 15`, `0 <= graph[i][j] < n`, `graph[i][j] != i`, the input is a DAG, all elements of `graph[i]` are distinct.

Example: `graph = [[1,2],[3],[3],[]]` → `[[0,1,3],[0,2,3]]`. Example: `graph = [[4,3,1],[3,2,4],[3],[4],[]]` → `[[0,4],[0,3,4],[0,1,3,4],[0,1,2,3,4],[0,1,4]]` (any order).

hint: Because the graph is acyclic, a plain DFS that records the current path will terminate; no visited set is needed.
hint: Extend the path node by node; whenever you reach node `n-1`, snapshot the current path into the answer, then backtrack.

```cpp
// starter
#include <vector>
std::vector<std::vector<int>> allPathsSourceTarget(std::vector<std::vector<int>>& graph);
```

```cpp
#include <functional>
std::vector<std::vector<int>> allPathsSourceTarget(std::vector<std::vector<int>>& graph) {
    int n = (int)graph.size();
    std::vector<std::vector<int>> res;
    std::vector<int> path{0};
    std::function<void(int)> dfs = [&](int u) {
        if (u == n - 1) { res.push_back(path); return; }
        for (int v : graph[u]) { path.push_back(v); dfs(v); path.pop_back(); }
    };
    dfs(0);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <algorithm>
using std::vector;
static vector<vector<int>> norm(vector<vector<int>> v) { std::sort(v.begin(), v.end()); return v; }
//__USER__
int main() {
    { vector<vector<int>> g{{1,2},{3},{3},{}};
      auto r = norm(allPathsSourceTarget(g));
      vector<vector<int>> e{{0,1,3},{0,2,3}};
      if (r != norm(e)) { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{4,3,1},{3,2,4},{3},{4},{}};
      auto r = norm(allPathsSourceTarget(g));
      vector<vector<int>> e{{0,4},{0,3,4},{0,1,3,4},{0,1,2,3,4},{0,1,4}};
      if (r != norm(e)) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{1},{}};
      auto r = norm(allPathsSourceTarget(g));
      vector<vector<int>> e{{0,1}};
      if (r != norm(e)) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{1,2,3},{3},{3},{}};
      auto r = norm(allPathsSourceTarget(g));
      vector<vector<int>> e{{0,1,3},{0,2,3},{0,3}};
      if (r != norm(e)) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Since the graph is a DAG, no cycle can trap the search, so a straightforward DFS with backtracking enumerates every source-to-target path. Keep a running `path` starting at node 0; on reaching node `n-1`, copy the path into the result. Recurse into each outgoing edge, pushing then popping the node to backtrack. The time is bounded by the number of paths times their length (exponential in the worst case but small for `n <= 15`); recursion depth is O(n).
