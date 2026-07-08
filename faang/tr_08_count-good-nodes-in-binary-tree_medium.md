## challenge: Count Good Nodes in Binary Tree
tags: tree, dfs, recursion
track: faang
difficulty: medium

Given the root of a binary tree, a node X is called *good* if on the path from the root to X there is no node with a value greater than X's value. Return the number of good nodes in the tree. The root is always good.

Constraints: `1 <= n <= 10^5` nodes, `-10^4 <= Node.val <= 10^4`.

Example: `[3,1,4,3,null,1,5]` → `4`. Example: `[3,3,null,4,2]` → `3`. Example: `[1]` → `1`.

hint: A node is good exactly when its value is at least the maximum value seen on the way down to it.
hint: Thread the running maximum of the current root-to-node path through the recursion as a parameter.
hint: At each node, count it if its value is greater than or equal to the running max, then recurse with an updated max into both children.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int goodNodes(TreeNode* root);
```

```cpp
int goodNodes(TreeNode* root) {
    std::function<int(TreeNode*, int)> dfs = [&](TreeNode* n, int mx) -> int {
        if (!n) return 0;
        int cnt = (n->val >= mx) ? 1 : 0;
        int nmx = std::max(mx, n->val);
        return cnt + dfs(n->left, nmx) + dfs(n->right, nmx);
    };
    return dfs(root, INT_MIN);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <optional>
#include <functional>
#include <algorithm>
#include <climits>
using std::vector;
using std::optional;
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
static TreeNode* build(const vector<optional<int>>& a) {
    if (a.empty() || !a[0].has_value()) return nullptr;
    TreeNode* root = new TreeNode(*a[0]);
    std::queue<TreeNode*> q; q.push(root);
    size_t i = 1;
    while (!q.empty() && i < a.size()) {
        TreeNode* cur = q.front(); q.pop();
        if (i < a.size()) { if (a[i].has_value()) { cur->left  = new TreeNode(*a[i]); q.push(cur->left);  } ++i; }
        if (i < a.size()) { if (a[i].has_value()) { cur->right = new TreeNode(*a[i]); q.push(cur->right); } ++i; }
    }
    return root;
}
//__USER__
int main() {
    using N = optional<int>;
    if (goodNodes(build({3,1,4,3,N{},1,5})) != 4) { std::puts("case1"); return 1; }
    if (goodNodes(build({3,3,N{},4,2}))     != 3) { std::puts("case2"); return 1; }
    if (goodNodes(build({1}))               != 1) { std::puts("case3"); return 1; }
    if (goodNodes(build({1,1,1})) != 3) { std::puts("case4"); return 1; }
    if (goodNodes(build({9,N{},3,6})) != 1) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A single pre-order DFS carries the maximum value encountered so far on the current root-to-node path. A node is good when its value is at least that maximum; we then descend with the possibly-updated maximum. Summing the per-node contributions yields the count. Each node is visited once. O(n) time, O(h) recursion depth.
