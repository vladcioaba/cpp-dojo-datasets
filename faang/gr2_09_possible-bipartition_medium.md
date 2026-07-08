## challenge: Possible Bipartition
tags: graph, bfs, coloring, bipartite
track: faang
difficulty: medium

We want to split a group of `n` people (labeled `1` to `n`) into two groups of any size. Each person may dislike some others, and they should not be placed into the same group as anyone they dislike. Given the integer `n` and an array `dislikes` where `dislikes[i] = [a, b]` indicates that person `a` and person `b` dislike each other, return `true` if and only if it is possible to split everyone into two groups this way.

Constraints: `1 <= n <= 2000`, `0 <= dislikes.length <= 10^4`, `dislikes[i].length == 2`, `1 <= a < b <= n`, all pairs distinct.

Example: `n = 4, dislikes = [[1,2],[1,3],[2,4]]` → `true` (`{1,4}` and `{2,3}`). Example: `n = 3, dislikes = [[1,2],[1,3],[2,3]]` → `false` (odd cycle of mutual dislikes).

hint: Build an undirected graph of dislikes; a valid split is exactly a 2-coloring of that graph.
hint: BFS/DFS each component, coloring neighbors the opposite color. A conflict (two disliking people forced into the same group) means the answer is `false`.

```cpp
// starter
#include <vector>
bool possibleBipartition(int n, std::vector<std::vector<int>>& dislikes);
```

```cpp
bool possibleBipartition(int n, std::vector<std::vector<int>>& dislikes) {
    std::vector<std::vector<int>> g(n + 1);
    for (auto& d : dislikes) { g[d[0]].push_back(d[1]); g[d[1]].push_back(d[0]); }
    std::vector<int> color(n + 1, 0);
    std::vector<int> stack;
    for (int i = 1; i <= n; ++i) {
        if (color[i] != 0) continue;
        color[i] = 1;
        stack.push_back(i);
        while (!stack.empty()) {
            int u = stack.back(); stack.pop_back();
            for (int v : g[u]) {
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
    { vector<vector<int>> d{{1,2},{1,3},{2,4}}; if (possibleBipartition(4, d) != true) { std::puts("case1"); return 1; } }
    { vector<vector<int>> d{{1,2},{1,3},{2,3}}; if (possibleBipartition(3, d) != false) { std::puts("case2"); return 1; } }
    { vector<vector<int>> d{{1,2},{2,3},{3,4},{4,5},{1,5}}; if (possibleBipartition(5, d) != false) { std::puts("case3"); return 1; } }
    { vector<vector<int>> d{}; if (possibleBipartition(3, d) != true) { std::puts("case4"); return 1; } }
    { vector<vector<int>> d{{1,2},{3,4},{5,6},{6,7},{8,9},{7,8}}; if (possibleBipartition(10, d) != true) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Each dislike is an undirected edge; a valid partition into two groups is precisely a proper 2-coloring of this graph, which exists iff the graph has no odd cycle. Traverse every component with BFS/DFS, assigning each newly seen neighbor the opposite color of the current node. If an already-colored neighbor shares the current node's color, an odd cycle exists and partition is impossible. Building the graph and traversing costs O(n + E) time and space.
