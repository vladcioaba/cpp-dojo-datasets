## challenge: Graph Valid Tree

tags: graph, union-find, dfs

track: faang
difficulty: medium

You are given `n` nodes labeled `0..n-1` and an undirected edge list `edges`. Return `true` if these edges form a valid tree — that is, the graph is fully connected and contains no cycle — and `false` otherwise.

Constraints: `1 <= n <= 2000`, `0 <= edges.length <= 5000`, `edges[i].length == 2`, `0 <= a, b < n`, no self-loops or duplicate edges.

Example: `n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]` → `true`. Example: `n = 5, edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]` → `false` (the edge `[1,3]` closes a cycle).

hint: A tree on `n` nodes has exactly two properties: it is connected and it has no cycle. Either one plus the right edge count implies the other.
hint: A tree with `n` nodes must have exactly `n - 1` edges — reject immediately if the count differs.
hint: With exactly `n - 1` edges, the graph is a tree iff no edge connects two nodes already in the same union-find set (no cycle) — which also forces full connectivity.

```cpp
// starter
#include <vector>
bool validTree(int n, std::vector<std::vector<int>>& edges);
```

```cpp
bool validTree(int n, std::vector<std::vector<int>>& edges) {
    if ((int)edges.size() != n - 1) return false;
    std::vector<int> parent(n);
    for (int i = 0; i < n; ++i) parent[i] = i;
    auto find = [&](int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    };
    for (auto& e : edges) {
        int a = find(e[0]), b = find(e[1]);
        if (a == b) return false;
        parent[a] = b;
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
    { vector<vector<int>> e{{0,1},{0,2},{0,3},{1,4}};         if (!validTree(5, e)) { std::puts("case1"); return 1; } }
    { vector<vector<int>> e{{0,1},{1,2},{2,3},{1,3},{1,4}};   if ( validTree(5, e)) { std::puts("case2"); return 1; } }
    { vector<vector<int>> e{{0,1},{2,3}};                     if ( validTree(4, e)) { std::puts("case3"); return 1; } }
    { vector<vector<int>> e{};                                if (!validTree(1, e)) { std::puts("case4"); return 1; } }
    { vector<vector<int>> e{{0,1}};                           if (!validTree(2, e)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A valid tree on `n` nodes has exactly `n - 1` edges, so reject any other count outright. Then run union-find: if any edge links two nodes that already share a root, it creates a cycle, so return `false`. Surviving `n - 1` edges with no cycle necessarily connect all nodes into one component, so the graph is a tree. O((n + E) * alpha(n)) time, O(n) space.
