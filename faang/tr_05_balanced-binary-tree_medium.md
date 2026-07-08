## challenge: Balanced Binary Tree
tags: tree, dfs, recursion
track: faang
difficulty: medium

Given the root of a binary tree, return `true` if it is height-balanced: for every node, the heights of its left and right subtrees differ by at most one.

Constraints: `0 <= n <= 5000` nodes, `-10^4 <= Node.val <= 10^4`.

Example: `[3,9,20,null,null,15,7]` → `true`. Example: `[1,2,2,3,3,null,null,4,4]` → `false`. Example: empty → `true`.

hint: A naive solution recomputes subtree heights repeatedly; fold the balance check into a single height computation instead.
hint: Do one post-order pass that returns each subtree's height, and record a failure the moment any node's two child heights differ by more than one.
hint: Use a shared flag (or a sentinel height like -1) so a single traversal both measures heights and reports whether the tree is balanced.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
bool isBalanced(TreeNode* root);
```

```cpp
bool isBalanced(TreeNode* root) {
    bool ok = true;
    std::function<int(TreeNode*)> height = [&](TreeNode* n) -> int {
        if (!n) return 0;
        int l = height(n->left);
        int r = height(n->right);
        if (l - r > 1 || r - l > 1) ok = false;
        return 1 + std::max(l, r);
    };
    height(root);
    return ok;
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
//__USER__
int main() {
    using N = optional<int>;
    if (!isBalanced(build({3,9,20,N{},N{},15,7})))       { std::puts("case1"); return 1; }
    if ( isBalanced(build({1,2,2,3,3,N{},N{},4,4})))     { std::puts("case2"); return 1; }
    if (!isBalanced(build({})))                          { std::puts("case3"); return 1; }
    if ( isBalanced(build({1,2,N{},3,N{},4})))           { std::puts("case4"); return 1; }
    if (!isBalanced(build({1,2,3})))                     { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A bottom-up post-order traversal returns every subtree's height exactly once and flips a shared flag whenever a node's two child heights differ by more than one. Because heights are never recomputed, the whole check is linear. O(n) time, O(h) recursion depth.
