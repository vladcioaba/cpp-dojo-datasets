## challenge: Subtree of Another Tree
tags: tree, dfs, recursion
track: faang
difficulty: medium

Given the roots of two binary trees `root` and `subRoot`, return `true` if there is a subtree of `root` that is identical in structure and node values to `subRoot`. A subtree of a node consists of that node together with all of its descendants.

Constraints: `1 <= n(root) <= 2000`, `1 <= n(subRoot) <= 1000`, `-10^4 <= Node.val <= 10^4`.

Example: `root = [3,4,5,1,2], subRoot = [4,1,2]` → `true`. Example: `root = [3,4,5,1,2,null,null,null,null,0], subRoot = [4,1,2]` → `false`. Example: `root = [1,1], subRoot = [1]` → `true`.

hint: You already know how to test whether two trees are identical — reuse that as a building block.
hint: Walk every node of `root` and, at each one, ask whether the subtree rooted there is identical to `subRoot`.
hint: `isSubtree(root, sub)` succeeds if `root` matches `sub` exactly, or if either child of `root` contains `sub`; an empty `sub` always matches.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
bool isSubtree(TreeNode* root, TreeNode* subRoot);
```

```cpp
static bool same(TreeNode* a, TreeNode* b) {
    if (!a && !b) return true;
    if (!a || !b) return false;
    return a->val == b->val && same(a->left, b->left) && same(a->right, b->right);
}
bool isSubtree(TreeNode* root, TreeNode* subRoot) {
    if (!subRoot) return true;
    if (!root) return false;
    return same(root, subRoot) || isSubtree(root->left, subRoot) || isSubtree(root->right, subRoot);
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
    using N = optional<int>;
    if (!isSubtree(build({3,4,5,1,2}), build({4,1,2})))                                { std::puts("case1"); return 1; }
    if ( isSubtree(build({3,4,5,1,2,N{},N{},N{},N{},0}), build({4,1,2})))              { std::puts("case2"); return 1; }
    if (!isSubtree(build({1,1}), build({1})))                                          { std::puts("case3"); return 1; }
    if ( isSubtree(build({1,2,3}), build({2,3})))                                      { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** For each node of the larger tree, run an identical-tree comparison against `subRoot`. The comparison matches when both nodes are null, and otherwise requires equal values with matching left and right subtrees. In the worst case every node triggers a full comparison, giving O(n * m) time and O(h) recursion depth.
