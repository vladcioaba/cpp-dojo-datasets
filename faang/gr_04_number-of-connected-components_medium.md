## challenge: Number of Connected Components in an Undirected Graph

tags: graph, union-find, dfs

track: faang
difficulty: medium

You have an undirected graph with `n` nodes labeled `0..n-1`, described by an edge list `edges` where `edges[i] = [a, b]` is an undirected edge between `a` and `b`. Return the number of connected components in the graph.

Constraints: `1 <= n <= 2000`, `0 <= edges.length <= 5000`, `edges[i].length == 2`, `0 <= a, b < n`, no self-loops or repeated edges.

Example: `n = 5, edges = [[0,1],[1,2],[3,4]]` → `2`. Example: `n = 5, edges = [[0,1],[1,2],[2,3],[3,4]]` → `1`.

hint: Every edge you add can only ever merge two components or do nothing — it never splits them.
hint: Start with `n` components and use union-find; each edge that joins two different sets reduces the count by one.
hint: An edge whose endpoints already share a root is redundant (it closes a cycle) and must not change the count.

```cpp
// starter
#include <vector>
int countComponents(int n, std::vector<std::vector<int>>& edges);
```

```cpp
int countComponents(int n, std::vector<std::vector<int>>& edges) {
    std::vector<int> parent(n);
    for (int i = 0; i < n; ++i) parent[i] = i;
    auto find = [&](int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    };
    int count = n;
    for (auto& e : edges) {
        int a = find(e[0]), b = find(e[1]);
        if (a != b) { parent[a] = b; --count; }
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> e{{0,1},{1,2},{3,4}};         if (countComponents(5, e) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> e{{0,1},{1,2},{2,3},{3,4}};   if (countComponents(5, e) != 1) { std::puts("case2"); return 1; } }
    { vector<vector<int>> e{};                          if (countComponents(4, e) != 4) { std::puts("case3"); return 1; } }
    { vector<vector<int>> e{};                          if (countComponents(1, e) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> e{{0,1},{1,2},{0,2}};         if (countComponents(3, e) != 1) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Union-find over `n` singletons: process each edge and, whenever it connects two different roots, merge them and decrement the running component count; edges inside an existing component (cycle-closing) leave the count unchanged. Path halving keeps each operation near-constant. O((n + E) * alpha(n)) time, O(n) space.
