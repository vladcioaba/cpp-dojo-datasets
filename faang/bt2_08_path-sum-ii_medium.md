## challenge: Path Sum II
tags: backtracking, tree, dfs
track: faang
difficulty: medium

Given the `root` of a binary tree and an integer `targetSum`, return all root-to-leaf paths where the sum of the node values along the path equals `targetSum`. Each path should be returned as the list of node values from the root down to the leaf, in that order. A leaf is a node with no children. Return the paths in any order.

Constraints: `0 <= n <= 5000` nodes, `-1000 <= Node.val <= 1000`, `-1000 <= targetSum <= 1000`.

Example: `[5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22` -> `[[5,4,11,2],[5,8,4,5]]`. Example: `[1,2,3], targetSum = 5` -> `[]`. Example: empty tree, `targetSum = 0` -> `[]`.

hint: Track the running path and the remaining target as you descend into the tree.
hint: Record the current path only when you reach a leaf and the remaining target has hit exactly zero.
hint: Pop the current node off the running path when you backtrack out of it so sibling branches start clean.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<std::vector<int>> pathSum(TreeNode* root, int targetSum);
```

```cpp
std::vector<std::vector<int>> pathSum(TreeNode* root, int targetSum) {
    std::vector<std::vector<int>> res;
    std::vector<int> cur;
    std::function<void(TreeNode*, int)> dfs = [&](TreeNode* node, int remain) {
        if (!node) return;
        cur.push_back(node->val);
        remain -= node->val;
        if (!node->left && !node->right) {
            if (remain == 0) res.push_back(cur);
        } else {
            dfs(node->left, remain);
            dfs(node->right, remain);
        }
        cur.pop_back();
    };
    dfs(root, targetSum);
    return res;
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
static vector<vector<int>> canon(vector<vector<int>> g) {
    std::sort(g.begin(), g.end());   // path order is meaningful; only reorder the list of paths
    return g;
}
//__USER__
int main() {
    using N = optional<int>;
    {
        auto root = build({5,4,8,11,N{},13,4,7,2,N{},N{},5,1});
        vector<vector<int>> want = {{5,4,11,2},{5,8,4,5}};
        if (canon(pathSum(root, 22)) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        auto root = build({1,2,3});
        if (!pathSum(root, 5).empty()) { std::puts("case2"); return 1; }
    }
    {
        if (!pathSum(build({}), 0).empty()) { std::puts("case3"); return 1; }
    }
    {
        auto root = build({1,2});
        vector<vector<int>> want = {{1,2}};
        if (canon(pathSum(root, 3)) != canon(want)) { std::puts("case4"); return 1; }
    }
    {
        auto root = build({-2,N{},-3});
        vector<vector<int>> want = {{-2,-3}};
        if (canon(pathSum(root, -5)) != canon(want)) { std::puts("case5"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** A depth-first walk carries the running path and the remaining target down the tree. At a leaf, a remaining target of zero means the accumulated path is an answer, so it is copied out. Popping the node on the way back up keeps the path correct for sibling branches. Every node is visited once and a valid path costs O(h) to copy, giving O(n * h) time in the worst case and O(h) recursion depth.
