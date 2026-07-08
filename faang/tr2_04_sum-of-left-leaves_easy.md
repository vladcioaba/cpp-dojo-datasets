## challenge: Sum of Left Leaves
tags: tree, dfs, recursion
track: faang
difficulty: easy

Given the root of a binary tree, return the sum of all left leaves. A left leaf is a leaf (a node with no children) that is the left child of its parent.

Constraints: `0 <= n <= 1000` nodes, `-1000 <= Node.val <= 1000`.

Example: `[3,9,20,null,null,15,7]` → `24` (the left leaves are `9` and `15`). Example: `[1]` → `0`. Example: empty → `0`.

hint: A node being a left child is a property the parent knows, not the node itself — so decide "is this a left leaf" when you look at a node's left child.
hint: When you descend into a left child that happens to be a leaf, add its value; otherwise keep recursing.
hint: Right children can still contain left leaves deeper down, so always recurse into the right subtree too — just never count the right child itself as a left leaf.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int sumOfLeftLeaves(TreeNode* root);
```

```cpp
int sumOfLeftLeaves(TreeNode* root) {
    if (!root) return 0;
    int sum = 0;
    if (root->left) {
        if (!root->left->left && !root->left->right) sum += root->left->val;
        else sum += sumOfLeftLeaves(root->left);
    }
    sum += sumOfLeftLeaves(root->right);
    return sum;
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
    if (sumOfLeftLeaves(build({3,9,20,N{},N{},15,7})) != 24) { std::puts("case1"); return 1; }
    if (sumOfLeftLeaves(build({1})) != 0)                    { std::puts("case2"); return 1; }
    if (sumOfLeftLeaves(build({})) != 0)                     { std::puts("case3"); return 1; }
    if (sumOfLeftLeaves(build({1,2,3,4,5})) != 4)            { std::puts("case4"); return 1; }
    if (sumOfLeftLeaves(build({-9,-3,2,N{},4,4,0,-6,N{},-5})) != -11) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Whether a node is a "left leaf" depends on how the parent reaches it, so we test the condition from the parent: when a node's left child is itself childless, add that child's value; otherwise recurse into it. We always recurse into the right subtree as well, since it can still contain left leaves deeper down. Each node is visited once. O(n) time, O(h) space for recursion.
