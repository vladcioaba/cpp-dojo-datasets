## challenge: Evaluate Division
tags: graph, bfs, dfs, weighted-graph
track: faang
difficulty: medium

You are given `equations` and `values`, where `equations[i] = [Ai, Bi]` and `values[i]` mean `Ai / Bi = values[i]`, with `Ai` and `Bi` being variable names (strings). You are also given `queries`, where `queries[j] = [Cj, Dj]` asks for the value of `Cj / Dj`. Return the answers to all queries. If a query cannot be determined from the given equations (an unknown variable appears, or the two variables are not connected), return `-1.0` for that query.

Constraints: `1 <= equations.length <= 20`, `1 <= queries.length <= 20`, `values[i] > 0`, variable names are lowercase strings of length up to 5. It is guaranteed no contradictions exist.

Example: `equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]` → `[6.0, 0.5, -1.0, 1.0, -1.0]`.

hint: Model each variable as a node; the equation `a/b = k` gives a directed edge `a→b` weighted `k` and `b→a` weighted `1/k`.
hint: A query `c/d` equals the product of edge weights along any path from `c` to `d`; search for such a path with BFS/DFS. Missing variables and disconnected pairs give `-1.0`.

```cpp
// starter
#include <vector>
#include <string>
std::vector<double> calcEquation(std::vector<std::vector<std::string>>& equations,
                                 std::vector<double>& values,
                                 std::vector<std::vector<std::string>>& queries);
```

```cpp
#include <unordered_map>
#include <queue>
#include <utility>
std::vector<double> calcEquation(std::vector<std::vector<std::string>>& equations,
                                 std::vector<double>& values,
                                 std::vector<std::vector<std::string>>& queries) {
    std::unordered_map<std::string, std::vector<std::pair<std::string, double>>> g;
    for (int i = 0; i < (int)equations.size(); ++i) {
        const std::string& a = equations[i][0];
        const std::string& b = equations[i][1];
        g[a].push_back({b, values[i]});
        g[b].push_back({a, 1.0 / values[i]});
    }
    std::vector<double> res;
    for (auto& q : queries) {
        const std::string& src = q[0];
        const std::string& dst = q[1];
        if (!g.count(src) || !g.count(dst)) { res.push_back(-1.0); continue; }
        std::unordered_map<std::string, double> vis;
        std::queue<std::pair<std::string, double>> bfs;
        bfs.push({src, 1.0}); vis[src] = 1.0;
        double ans = -1.0;
        while (!bfs.empty()) {
            auto [cur, val] = bfs.front(); bfs.pop();
            if (cur == dst) { ans = val; break; }
            for (auto& [nx, w] : g[cur]) {
                if (!vis.count(nx)) { vis[nx] = val * w; bfs.push({nx, val * w}); }
            }
        }
        res.push_back(ans);
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <cmath>
using std::vector;
using std::string;
static bool close(double a, double b) { return std::fabs(a - b) < 1e-6; }
//__USER__
int main() {
    {
        vector<vector<string>> eq{{"a","b"},{"b","c"}};
        vector<double> val{2.0, 3.0};
        vector<vector<string>> qr{{"a","c"},{"b","a"},{"a","e"},{"a","a"},{"x","x"}};
        auto r = calcEquation(eq, val, qr);
        vector<double> e{6.0, 0.5, -1.0, 1.0, -1.0};
        if (r.size() != e.size()) { std::puts("case1-size"); return 1; }
        for (size_t i = 0; i < e.size(); ++i) if (!close(r[i], e[i])) { std::puts("case1"); return 1; }
    }
    {
        vector<vector<string>> eq{{"a","b"},{"b","c"},{"bc","cd"}};
        vector<double> val{1.5, 2.5, 5.0};
        vector<vector<string>> qr{{"a","c"},{"c","b"},{"bc","cd"},{"cd","bc"}};
        auto r = calcEquation(eq, val, qr);
        vector<double> e{3.75, 0.4, 5.0, 0.2};
        for (size_t i = 0; i < e.size(); ++i) if (!close(r[i], e[i])) { std::puts("case2"); return 1; }
    }
    {
        vector<vector<string>> eq{{"a","b"}};
        vector<double> val{0.5};
        vector<vector<string>> qr{{"a","b"},{"b","a"},{"a","c"},{"x","y"}};
        auto r = calcEquation(eq, val, qr);
        vector<double> e{0.5, 2.0, -1.0, -1.0};
        for (size_t i = 0; i < e.size(); ++i) if (!close(r[i], e[i])) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Build a weighted directed graph where `a/b = k` adds edges `a→b` (weight `k`) and `b→a` (weight `1/k`). The value of a query `c/d` is the product of edge weights along any path from `c` to `d`, since divisions compose multiplicatively. For each query, if either variable is unknown, answer `-1.0`; otherwise BFS/DFS from `c` accumulating the product, returning the value at `d` or `-1.0` if unreachable. With V variables and Q queries the cost is O(Q * (V + E)).
