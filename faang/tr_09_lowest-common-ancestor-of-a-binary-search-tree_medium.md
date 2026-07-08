## challenge: Lowest Common Ancestor of a Binary Search Tree
tags: tree, dfs, binary-search-tree
track: faang
difficulty: medium

Given the root of a binary search tree and two nodes `p` and `q` present in it, return their lowest common ancestor (LCA): the deepest node that has both `p` and `q` as descendants, where a node is allowed to be a descendant of itself. All node values are unique.

Constraints: `2 <= n <= 10^5` nodes, `-10^9 <= Node.val <= 10^9`, `p != q`.

Example: `root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8` → `6`. Example: same tree, `p = 2, q = 4` → `2`. Example: `p = 3, q = 5` → `4`.

hint: The BST ordering tells you which direction each target lies without exploring both subtrees.
hint: If both `p` and `q` are smaller than the current node, the LCA is in the left subtree; if both are larger, it is in the right subtree.
hint: The first node where the two targets diverge — one on each side, or one equal to the node — is the lowest common ancestor.

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
    TreeNode* cur = root;
    while (cur) {
        if (p->val < cur->val && q->val < cur->val)      cur = cur->left;
        else if (p->val > cur->val && q->val > cur->val) cur = cur->right;
        else return cur;
    }
    return nullptr;
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
    while (r) { if (v == r->val) return r; r = (v < r->val) ? r->left : r->right; }
    return nullptr;
}
//__USER__
int main() {
    using N = optional<int>;
    TreeNode* t = build({6,2,8,0,4,7,9,N{},N{},3,5});
    if (lowestCommonAncestor(t, find(t,2), find(t,8))->val != 6) { std::puts("case1"); return 1; }
    if (lowestCommonAncestor(t, find(t,2), find(t,4))->val != 2) { std::puts("case2"); return 1; }
    if (lowestCommonAncestor(t, find(t,3), find(t,5))->val != 4) { std::puts("case3"); return 1; }
    if (lowestCommonAncestor(t, find(t,0), find(t,5))->val != 2) { std::puts("case4"); return 1; }
    if (lowestCommonAncestor(t, find(t,7), find(t,9))->val != 8) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Exploit the BST invariant: from the current node, if both targets are smaller go left, if both are larger go right, otherwise the targets split here (or one equals the node) and this node is the lowest common ancestor. The walk follows a single root-to-node path. O(h) time, O(1) space.
