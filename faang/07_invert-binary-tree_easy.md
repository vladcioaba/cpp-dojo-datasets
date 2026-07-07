## challenge: Invert Binary Tree
tags: tree, dfs, recursion
track: faang
difficulty: easy

Given the root of a binary tree, invert it — swap the left and right child of every node — and return the root.

Constraints: `0 <= n <= 100` nodes, `-100 <= Node.val <= 100`.

Example: `[4,2,7,1,3,6,9]` → `[4,7,2,9,6,3,1]` (level-order, `null` for missing children). Example: empty → empty.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
TreeNode* invertTree(TreeNode* root);
```

```cpp
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    TreeNode* l = invertTree(root->left);
    TreeNode* r = invertTree(root->right);
    root->left = r;
    root->right = l;
    return root;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
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
static vector<optional<int>> toLevel(TreeNode* root) {
    vector<optional<int>> out;
    if (!root) return out;
    std::queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* c = q.front(); q.pop();
        if (c) { out.push_back(c->val); q.push(c->left); q.push(c->right); }
        else out.push_back(std::nullopt);
    }
    while (!out.empty() && !out.back().has_value()) out.pop_back();
    return out;
}
//__USER__
int main() {
    auto r1 = toLevel(invertTree(build({4,2,7,1,3,6,9})));
    if (r1 != vector<optional<int>>({4,7,2,9,6,3,1})) { std::puts("case1"); return 1; }
    auto r2 = toLevel(invertTree(build({2,1,3})));
    if (r2 != vector<optional<int>>({2,3,1})) { std::puts("case2"); return 1; }
    auto r3 = toLevel(invertTree(build({})));
    if (!r3.empty()) { std::puts("case3"); return 1; }
    std::puts("PASS");
}
```
