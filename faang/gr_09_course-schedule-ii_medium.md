## challenge: Course Schedule II

tags: graph, topological-sort, bfs, cycle-detection

track: faang
difficulty: medium

There are `numCourses` courses labeled `0..numCourses-1`. `prerequisites[i] = [a, b]` means you must take course `b` before course `a`. Return any ordering of courses you can follow to finish them all. If it is impossible (the dependencies contain a cycle), return an empty array.

Constraints: `1 <= numCourses <= 2000`, `0 <= prerequisites.length <= numCourses * (numCourses - 1)`, `prerequisites[i].length == 2`, `0 <= a, b < numCourses`, pairs are distinct.

Example: `numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]` → e.g. `[0,1,2,3]` (any valid topological order). Example: `numCourses = 2, prerequisites = [[0,1],[1,0]]` → `[]` (cycle).

hint: A valid ordering is a topological sort of the prerequisite graph; no ordering exists exactly when the graph has a cycle.
hint: Kahn's algorithm: repeatedly take a course with no remaining prerequisites, output it, and relax its dependents' in-degrees.
hint: If you output fewer than `numCourses` courses, a cycle blocked the rest — return an empty array.

```cpp
// starter
#include <vector>
std::vector<int> findOrder(int numCourses, std::vector<std::vector<int>>& prerequisites);
```

```cpp
std::vector<int> findOrder(int numCourses, std::vector<std::vector<int>>& prerequisites) {
    std::vector<std::vector<int>> adj(numCourses);
    std::vector<int> indeg(numCourses, 0);
    for (auto& p : prerequisites) { adj[p[1]].push_back(p[0]); ++indeg[p[0]]; }
    std::queue<int> q;
    for (int i = 0; i < numCourses; ++i) if (indeg[i] == 0) q.push(i);
    std::vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u]) if (--indeg[v] == 0) q.push(v);
    }
    if ((int)order.size() != numCourses) return {};
    return order;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
using std::vector;
static bool validOrder(int n, vector<vector<int>>& prereq, const vector<int>& order) {
    if ((int)order.size() != n) return false;
    vector<int> pos(n, -1);
    for (int i = 0; i < n; ++i) {
        int c = order[i];
        if (c < 0 || c >= n || pos[c] != -1) return false;
        pos[c] = i;
    }
    for (auto& p : prereq) if (pos[p[1]] > pos[p[0]]) return false;
    return true;
}
//__USER__
int main() {
    { vector<vector<int>> p{{1,0}};                     auto r = findOrder(2, p); if (!validOrder(2, p, r)) { std::puts("case1"); return 1; } }
    { vector<vector<int>> p{{1,0},{2,0},{3,1},{3,2}};   auto r = findOrder(4, p); if (!validOrder(4, p, r)) { std::puts("case2"); return 1; } }
    { vector<vector<int>> p{};                          auto r = findOrder(1, p); if (!validOrder(1, p, r)) { std::puts("case3"); return 1; } }
    { vector<vector<int>> p{{0,1},{1,0}};               auto r = findOrder(2, p); if (!r.empty())          { std::puts("case4"); return 1; } }
    { vector<vector<int>> p{{1,0},{1,2},{0,1}};         auto r = findOrder(3, p); if (!r.empty())          { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Build the directed graph and in-degree array, then run Kahn's topological sort: seed a queue with every zero-in-degree course, and each time you dequeue a course append it to the order and decrement its neighbours' in-degrees, enqueuing any that reach zero. If the produced order covers all courses it is a valid schedule; otherwise a cycle left some courses unprocessed and no schedule exists, so return empty. O(V + E) time, O(V + E) space.
