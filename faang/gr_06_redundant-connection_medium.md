## challenge: Redundant Connection

tags: graph, union-find, cycle-detection

track: faang
difficulty: medium

A tree is an undirected graph that is connected and acyclic. You start with a tree of `n` nodes labeled `1..n` and add one extra edge, producing exactly one cycle. Given the resulting `edges` list (each `edges[i] = [a, b]`), return the edge that can be removed so the remaining graph is again a tree with `n` nodes. If several answers exist, return the one that appears last in `edges`.

Constraints: `n == edges.length`, `3 <= n <= 1000`, `edges[i].length == 2`, `1 <= a < b <= n`, the input is a tree plus exactly one additional edge.

Example: `edges = [[1,2],[1,3],[2,3]]` → `[2,3]`. Example: `edges = [[1,2],[2,3],[3,4],[1,4],[1,5]]` → `[1,4]`.

hint: Adding the extra edge creates exactly one cycle; the answer is an edge whose two endpoints were already connected before that edge was processed.
hint: Process edges in input order with union-find; the first edge whose endpoints already share a root is the one that closes the cycle.
hint: Because you scan in order, that first cycle-closing edge is automatically the last valid removable edge for this "tree plus one" input.

```cpp
// starter
#include <vector>
std::vector<int> findRedundantConnection(std::vector<std::vector<int>>& edges);
```

```cpp
std::vector<int> findRedundantConnection(std::vector<std::vector<int>>& edges) {
    int n = (int)edges.size();
    std::vector<int> parent(n + 1);
    for (int i = 0; i <= n; ++i) parent[i] = i;
    auto find = [&](int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    };
    for (auto& e : edges) {
        int a = find(e[0]), b = find(e[1]);
        if (a == b) return e;
        parent[a] = b;
    }
    return {};
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> e{{1,2},{1,3},{2,3}};               auto r = findRedundantConnection(e); if (r != vector<int>({2,3})) { std::puts("case1"); return 1; } }
    { vector<vector<int>> e{{1,2},{2,3},{3,4},{1,4},{1,5}};   auto r = findRedundantConnection(e); if (r != vector<int>({1,4})) { std::puts("case2"); return 1; } }
    { vector<vector<int>> e{{1,2},{2,3},{1,3}};               auto r = findRedundantConnection(e); if (r != vector<int>({1,3})) { std::puts("case3"); return 1; } }
    { vector<vector<int>> e{{1,4},{3,4},{1,3},{1,2}};         auto r = findRedundantConnection(e); if (r != vector<int>({1,3})) { std::puts("case4"); return 1; } }
    { vector<vector<int>> e{{2,3},{1,2},{1,3},{4,1}};         auto r = findRedundantConnection(e); if (r != vector<int>({1,3})) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Scan the edges in order and union their endpoints. The moment an edge's two endpoints already share a root, that edge would create the graph's single cycle, so it is the redundant one; return it. Scanning left to right guarantees this is the last removable edge for a tree-plus-one-edge input. O(n * alpha(n)) time, O(n) space.
