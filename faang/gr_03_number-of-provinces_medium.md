## challenge: Number of Provinces

tags: graph, union-find, dfs

track: faang
difficulty: medium

There are `n` cities labeled `0..n-1`. You are given an `n x n` matrix `isConnected` where `isConnected[i][j] = 1` means cities `i` and `j` are directly connected (and `isConnected[i][i] = 1`). A province is a group of cities that are directly or indirectly connected. Return the number of provinces.

Constraints: `1 <= n <= 200`, `isConnected[i][j]` is `0` or `1`, `isConnected[i][j] == isConnected[j][i]`, `isConnected[i][i] == 1`.

Example: `isConnected = [[1,1,0],[1,1,0],[0,0,1]]` → `2`. Example: `isConnected = [[1,0,0],[0,1,0],[0,0,1]]` → `3`.

hint: A province is just a connected component of the city graph — count the components.
hint: Union-find (disjoint set union) starts with `n` singletons and merges two whenever they are directly connected.
hint: Begin the answer at `n` and decrement it every time a `union` actually joins two previously separate sets.

```cpp
// starter
#include <vector>
int findCircleNum(std::vector<std::vector<int>>& isConnected);
```

```cpp
int findCircleNum(std::vector<std::vector<int>>& isConnected) {
    int n = (int)isConnected.size();
    std::vector<int> parent(n);
    for (int i = 0; i < n; ++i) parent[i] = i;
    auto find = [&](int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    };
    int count = n;
    for (int i = 0; i < n; ++i)
        for (int j = i + 1; j < n; ++j)
            if (isConnected[i][j]) {
                int a = find(i), b = find(j);
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
    { vector<vector<int>> m{{1,1,0},{1,1,0},{0,0,1}}; if (findCircleNum(m) != 2) { std::puts("case1"); return 1; } }
    { vector<vector<int>> m{{1,0,0},{0,1,0},{0,0,1}}; if (findCircleNum(m) != 3) { std::puts("case2"); return 1; } }
    { vector<vector<int>> m{{1}};                     if (findCircleNum(m) != 1) { std::puts("case3"); return 1; } }
    { vector<vector<int>> m{{1,1,1},{1,1,1},{1,1,1}}; if (findCircleNum(m) != 1) { std::puts("case4"); return 1; } }
    { vector<vector<int>> m{{1,0,0,1},{0,1,1,0},{0,1,1,0},{1,0,0,1}}; if (findCircleNum(m) != 2) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Treat the adjacency matrix as an undirected graph and count connected components with union-find. Start with `n` disjoint singletons; for every directly connected pair `(i, j)`, union their sets and, when the union merges two distinct roots, decrement the component count. With path halving the near-flat inverse-Ackermann cost gives O(n^2 * alpha(n)) time (dominated by scanning the matrix), O(n) space.
