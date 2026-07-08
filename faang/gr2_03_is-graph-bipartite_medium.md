## challenge: Is Graph Bipartite
tags: graph, bfs, coloring
track: faang
difficulty: medium

There is an undirected graph with `n` nodes labeled `0` to `n - 1`, given as an adjacency list `graph`, where `graph[u]` is the list of nodes adjacent to `u`. The graph has no self-edges and no parallel edges, and it may be disconnected. Return `true` if and only if the graph is bipartite: the nodes can be split into two disjoint sets `A` and `B` such that every edge connects a node in `A` with a node in `B`.

Constraints: `1 <= n <= 100`, `0 <= graph[u].length < n`, edges are symmetric, no self-loops.

Example: `graph = [[1,3],[0,2],[1,3],[0,2]]` → `true` (`{0,2}` and `{1,3}`). Example: `graph = [[1,2,3],[0,2],[0,1,3],[0,2]]` → `false` (contains an odd cycle 0-1-2-0).

hint: A graph is bipartite exactly when it contains no odd-length cycle. Try to two-color it.
hint: BFS/DFS each component, assigning alternating colors. If you ever need to give a node the same color as an adjacent node, it is not bipartite.

```cpp
// starter
#include <vector>
bool isBipartite(std::vector<std::vector<int>>& graph);
```

```cpp
bool isBipartite(std::vector<std::vector<int>>& graph) {
    int n = (int)graph.size();
    std::vector<int> color(n, 0);
    std::vector<int> stack;
    for (int i = 0; i < n; ++i) {
        if (color[i] != 0) continue;
        color[i] = 1;
        stack.push_back(i);
        while (!stack.empty()) {
            int u = stack.back(); stack.pop_back();
            for (int v : graph[u]) {
                if (color[v] == 0) { color[v] = -color[u]; stack.push_back(v); }
                else if (color[v] == color[u]) return false;
            }
        }
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
    { vector<vector<int>> g{{1,3},{0,2},{1,3},{0,2}}; if (isBipartite(g) != true) { std::puts("case1"); return 1; } }
    { vector<vector<int>> g{{1,2,3},{0,2},{0,1,3},{0,2}}; if (isBipartite(g) != false) { std::puts("case2"); return 1; } }
    { vector<vector<int>> g{{}}; if (isBipartite(g) != true) { std::puts("case3"); return 1; } }
    { vector<vector<int>> g{{1},{0},{3},{2}}; if (isBipartite(g) != true) { std::puts("case4"); return 1; } }
    { vector<vector<int>> g{{1,2},{0,2},{0,1}}; if (isBipartite(g) != false) { std::puts("case5"); return 1; } }
    { vector<vector<int>> g{{},{},{}}; if (isBipartite(g) != true) { std::puts("case6"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** A graph is bipartite iff it has no odd cycle, which is equivalent to being 2-colorable. Iterate over every node to handle disconnected components; for each uncolored node, color it and traverse (BFS or DFS), assigning each neighbor the opposite color. If a neighbor already carries the same color as the current node, an odd cycle exists and we return `false`. Each node and edge is visited once: O(n + E) time, O(n) space.
