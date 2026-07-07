## challenge: Course Schedule
tags: graph, topological-sort, cycle-detection, bfs
track: faang
difficulty: medium

There are `numCourses` courses labeled `0..numCourses-1`. `prerequisites[i] = [a, b]` means you must take course `b` before course `a`. Return `true` if you can finish all courses (i.e. the dependency graph is acyclic), otherwise `false`.

Constraints: `1 <= numCourses <= 2000`, `0 <= prerequisites.length <= 5000`, pairs may be distinct.

Example: `numCourses = 2, prerequisites = [[1,0]]` → `true`. Example: `[[1,0],[0,1]]` → `false` (cycle).

hint: "Can I finish?" is really asking whether the prerequisite graph contains a cycle.
hint: Topological sort — Kahn's algorithm repeatedly removes nodes that have no remaining prerequisites.
hint: Track in-degrees, enqueue every zero-in-degree node, and if you cannot process all N courses a cycle exists.

```cpp
// starter
#include <vector>
bool canFinish(int numCourses, std::vector<std::vector<int>>& prerequisites);
```

```cpp
bool canFinish(int numCourses, std::vector<std::vector<int>>& prerequisites) {
    std::vector<std::vector<int>> adj(numCourses);
    std::vector<int> indeg(numCourses, 0);
    for (auto& p : prerequisites) {
        adj[p[1]].push_back(p[0]);
        ++indeg[p[0]];
    }
    std::queue<int> q;
    for (int i = 0; i < numCourses; ++i) if (indeg[i] == 0) q.push(i);
    int seen = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        ++seen;
        for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
    }
    return seen == numCourses;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
using std::vector;
//__USER__
int main() {
    { vector<vector<int>> p{{1,0}};             if (!canFinish(2, p)) { std::puts("case1"); return 1; } }
    { vector<vector<int>> p{{1,0},{0,1}};       if ( canFinish(2, p)) { std::puts("case2"); return 1; } }
    { vector<vector<int>> p{};                  if (!canFinish(1, p)) { std::puts("case3"); return 1; } }
    { vector<vector<int>> p{{1,0},{2,1},{3,2}}; if (!canFinish(4, p)) { std::puts("case4"); return 1; } }
    { vector<vector<int>> p{{0,1},{1,2},{2,0}}; if ( canFinish(3, p)) { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Model courses as a directed graph and run Kahn's topological sort: repeatedly take a node of in-degree 0 and decrement its neighbors' in-degrees. If every node gets processed there is no cycle, so all courses are finishable. O(V+E) time, O(V+E) space.
