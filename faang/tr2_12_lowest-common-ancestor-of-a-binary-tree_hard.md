## challenge: Lowest Common Ancestor of a Binary Tree
tags: tree, dfs, recursion
track: faang
difficulty: hard

Given the root of a binary tree and two nodes `p` and `q` present in it, return their lowest common ancestor (LCA): the deepest node that has both `p` and `q` as descendants, where a node is allowed to be a descendant of itself. Unlike the BST version, there is no ordering to exploit. All node values are unique.

Constraints: `2 <= n <= 10^5` nodes, `-10^9 <= Node.val <= 10^9`, `p != q`, both nodes exist in the tree.

Example: `root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1` → `3`. Example: same tree, `p = 5, q = 4` → `5` (a node can be an ancestor of itself). Example: `p = 7, q = 4` → `2`.

hint: Without an ordering, you must actually search both subtrees rather than choosing a direction.
hint: Recurse: a call returns non-null if `p` or `q` (or their LCA) is found somewhere in that subtree.
hint: If a node finds one target in its left subtree and the other in its right subtree, that node is the LCA; if both come from the same side, propagate that side's answer upward.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q);
```

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);
    if (left && right) return root;
    return left ? left : right;
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
static TreeNode* find(TreeNode* r, int v) {
    if (!r) return nullptr;
    if (r->val == v) return r;
    TreeNode* l = find(r->left, v);
    return l ? l : find(r->right, v);
}
//__USER__
int main() {
    using N = optional<int>;
    TreeNode* t = build({3,5,1,6,2,0,8,N{},N{},7,4});
    if (lowestCommonAncestor(t, find(t,5), find(t,1))->val != 3) { std::puts("case1"); return 1; }
    if (lowestCommonAncestor(t, find(t,5), find(t,4))->val != 5) { std::puts("case2"); return 1; }
    if (lowestCommonAncestor(t, find(t,7), find(t,4))->val != 2) { std::puts("case3"); return 1; }
    if (lowestCommonAncestor(t, find(t,6), find(t,8))->val != 3) { std::puts("case4"); return 1; }
    if (lowestCommonAncestor(t, find(t,7), find(t,8))->val != 3) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Recurse over the whole tree since there is no ordering to prune with. Each call returns a non-null pointer when its subtree contains `p`, `q`, or the ancestor joining them. A node whose left and right subtrees each surface one of the targets is their lowest common ancestor; if both come back from the same side, that side's result bubbles up unchanged. The self-descendant rule is handled by returning a node immediately when it equals `p` or `q`. Each node is visited once. O(n) time, O(h) space for recursion.
