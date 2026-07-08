## challenge: Symmetric Tree
tags: tree, dfs, recursion
track: faang
difficulty: easy

Given the root of a binary tree, return `true` if it is a mirror image of itself around its center — that is, the left subtree is the mirror reflection of the right subtree.

Constraints: `1 <= n <= 1000` nodes, `-100 <= Node.val <= 100`.

Example: `[1,2,2,3,4,4,3]` → `true`. Example: `[1,2,2,null,3,null,3]` → `false`. Example: `[1]` → `true`.

hint: Symmetry is not about one subtree in isolation — it is a relationship between the left and right subtrees.
hint: Two trees mirror each other when their roots match and the left of one mirrors the right of the other, and vice versa.
hint: Write a helper `mirror(a, b)`: both null is a match, exactly one null is a mismatch, otherwise compare values and recurse on `(a->left, b->right)` and `(a->right, b->left)`.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
bool isSymmetric(TreeNode* root);
```

```cpp
static bool mirror(TreeNode* a, TreeNode* b) {
    if (!a && !b) return true;
    if (!a || !b) return false;
    return a->val == b->val && mirror(a->left, b->right) && mirror(a->right, b->left);
}
bool isSymmetric(TreeNode* root) {
    return !root || mirror(root->left, root->right);
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
//__USER__
int main() {
    if (!isSymmetric(build({1,2,2,3,4,4,3})))                       { std::puts("case1"); return 1; }
    if ( isSymmetric(build({1,2,2,std::nullopt,3,std::nullopt,3}))) { std::puts("case2"); return 1; }
    if (!isSymmetric(build({1})))                                   { std::puts("case3"); return 1; }
    if ( isSymmetric(build({1,2,3})))                               { std::puts("case4"); return 1; }
    if (!isSymmetric(build({1,2,2,std::nullopt,3,3,std::nullopt}))) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Reduce the whole-tree question to a pairwise mirror check between two nodes. Two subtrees are mirrors when both roots are absent, or both are present with equal values and the outer/inner child pairs mirror each other recursively. Each node is compared once. O(n) time, O(h) recursion depth.
