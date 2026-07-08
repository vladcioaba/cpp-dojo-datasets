## challenge: Path Sum II
tags: tree, dfs, backtracking
track: faang
difficulty: medium

Given the root of a binary tree and an integer `targetSum`, return all root-to-leaf paths where the sum of the node values along the path equals `targetSum`. Each path is returned as the list of node values in order from root to leaf. A leaf is a node with no children.

Constraints: `0 <= n <= 5000` nodes, `-1000 <= Node.val <= 1000`, `-10^9 <= targetSum <= 10^9`.

Example: `[5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22` → `[[5,4,11,2],[5,8,4,5]]`. Example: `[1,2,3], targetSum = 5` → `[]`. Example: empty, `targetSum = 0` → `[]`.

hint: This is a depth-first walk that must remember the whole path so far, not just a running sum.
hint: Push the current node onto a path list on entry and pop it on exit — classic backtracking so siblings do not see each other's nodes.
hint: Record a copy of the path only when you are at a leaf and the remaining target has reached exactly zero.

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
    std::vector<int> path;
    std::function<void(TreeNode*, int)> dfs = [&](TreeNode* node, int rem) {
        if (!node) return;
        path.push_back(node->val);
        rem -= node->val;
        if (!node->left && !node->right) {
            if (rem == 0) res.push_back(path);
        } else {
            dfs(node->left, rem);
            dfs(node->right, rem);
        }
        path.pop_back();
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
#include <functional>
#include <optional>
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
    {
        auto r = pathSum(build({5,4,8,11,N{},13,4,7,2,N{},N{},5,1}), 22);
        vector<vector<int>> want = {{5,4,11,2},{5,8,4,5}};
        if (r != want) { std::puts("case1"); return 1; }
    }
    {
        if (!pathSum(build({1,2,3}), 5).empty()) { std::puts("case2"); return 1; }
    }
    {
        if (!pathSum(build({}), 0).empty()) { std::puts("case3"); return 1; }
    }
    {
        auto r = pathSum(build({1,2}), 1);
        if (!r.empty()) { std::puts("case4"); return 1; }
    }
    {
        auto r = pathSum(build({-2,N{},-3}), -5);
        vector<vector<int>> want = {{-2,-3}};
        if (r != want) { std::puts("case5"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Depth-first search carrying both the running path and the remaining target. On entering a node we append it and subtract its value; at a leaf we save a copy of the path only if the remainder is zero; on exit we pop to backtrack so unrelated branches never share nodes. Building each qualifying path costs its length. O(n^2) worst case for copying paths, O(h) auxiliary space.
