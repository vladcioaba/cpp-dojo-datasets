## challenge: Clone Graph

tags: graph, dfs, bfs, hash-table

track: faang
difficulty: medium

Given a reference to a node in a connected undirected graph, return a deep copy (clone) of the graph. Each `Node` holds an integer `val` and a list of its neighbours. The clone must consist of entirely new `Node` objects with the same values and the same neighbour structure — no pointer may be shared with the original. If the input is `nullptr`, return `nullptr`.

Constraints: `0 <= number of nodes <= 100`, `1 <= Node.val <= 100`, values are unique, the graph is undirected with no self-loops or repeated edges and is connected.

Example: a 4-node graph where node 1 neighbours {2,4}, node 2 neighbours {1,3}, node 3 neighbours {2,4}, node 4 neighbours {1,3} → an isomorphic graph of fresh nodes. Example: `nullptr` → `nullptr`.

hint: The hard part is not the traversal — it is making sure each original node maps to exactly one freshly-allocated copy, even across cycles.
hint: Keep a hash map from original node to its clone; consult it before creating a copy so shared/cyclic neighbours reuse the same clone.
hint: DFS (or BFS): create the clone, record it in the map immediately, then recurse to wire up cloned neighbours.

```cpp
// starter
#include <vector>
struct Node {
    int val;
    std::vector<Node*> neighbors;
    Node() : val(0) {}
    Node(int v) : val(v) {}
};
Node* cloneGraph(Node* node);
```

```cpp
Node* cloneGraph(Node* node) {
    if (!node) return nullptr;
    std::unordered_map<Node*, Node*> clones;
    std::function<Node*(Node*)> dfs = [&](Node* cur) -> Node* {
        auto it = clones.find(cur);
        if (it != clones.end()) return it->second;
        Node* copy = new Node(cur->val);
        clones[cur] = copy;
        for (Node* nb : cur->neighbors) copy->neighbors.push_back(dfs(nb));
        return copy;
    };
    return dfs(node);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <unordered_set>
#include <queue>
#include <functional>
#include <algorithm>
using std::vector;
struct Node {
    int val;
    std::vector<Node*> neighbors;
    Node() : val(0) {}
    Node(int v) : val(v) {}
};
static Node* buildGraph(const vector<vector<int>>& adj) {
    int n = (int)adj.size();
    if (n == 0) return nullptr;
    vector<Node*> nodes(n + 1, nullptr);
    for (int i = 1; i <= n; ++i) nodes[i] = new Node(i);
    for (int i = 1; i <= n; ++i)
        for (int nb : adj[i - 1]) nodes[i]->neighbors.push_back(nodes[nb]);
    return nodes[1];
}
static void collectPtrs(Node* start, std::unordered_set<Node*>& seen) {
    if (!start) return;
    std::queue<Node*> q; q.push(start); seen.insert(start);
    while (!q.empty()) {
        Node* u = q.front(); q.pop();
        for (Node* v : u->neighbors) if (!seen.count(v)) { seen.insert(v); q.push(v); }
    }
}
static vector<vector<int>> toAdj(Node* start, int n) {
    vector<vector<int>> res(n);
    if (!start) return res;
    std::unordered_set<Node*> seen; std::queue<Node*> q; q.push(start); seen.insert(start);
    while (!q.empty()) {
        Node* u = q.front(); q.pop();
        vector<int> nb;
        for (Node* v : u->neighbors) { nb.push_back(v->val); if (!seen.count(v)) { seen.insert(v); q.push(v); } }
        std::sort(nb.begin(), nb.end());
        if (u->val - 1 >= 0 && u->val - 1 < n) res[u->val - 1] = nb;
    }
    return res;
}
//__USER__
static bool check(const vector<vector<int>>& adj) {
    int n = (int)adj.size();
    Node* orig = buildGraph(adj);
    std::unordered_set<Node*> origPtrs;
    collectPtrs(orig, origPtrs);
    Node* cl = cloneGraph(orig);
    if (n == 0) return cl == nullptr;
    if (toAdj(cl, n) != adj) return false;
    std::unordered_set<Node*> clonePtrs;
    collectPtrs(cl, clonePtrs);
    if ((int)clonePtrs.size() != n) return false;
    for (Node* p : clonePtrs) if (origPtrs.count(p)) return false;
    return true;
}
int main() {
    if (!check({{2,4},{1,3},{2,4},{1,3}})) { std::puts("case1"); return 1; }
    if (!check({{}}))                      { std::puts("case2"); return 1; }
    if (!check({}))                        { std::puts("case3"); return 1; }
    if (!check({{2},{1}}))                 { std::puts("case4"); return 1; }
    if (!check({{2,3},{1,3},{1,2}}))       { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Traverse the graph once while maintaining a map from each original node to its freshly-allocated clone. On visiting a node, if it is already in the map return the stored clone; otherwise allocate the copy, register it *before* recursing so cycles terminate, then recursively clone and attach each neighbour. This produces a structurally identical graph of all-new nodes. O(V + E) time, O(V) space for the map and recursion.
