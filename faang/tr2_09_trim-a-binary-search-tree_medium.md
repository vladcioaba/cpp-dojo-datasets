## challenge: Trim a Binary Search Tree
tags: tree, binary-search-tree, recursion
track: faang
difficulty: medium

Given the root of a binary search tree and a range `[low, high]`, trim the tree so that all its node values lie within `[low, high]` inclusive. Trimming should keep the relative structure of the remaining nodes intact; a node stays in the tree if and only if its value is in range. Return the root of the trimmed tree.

Constraints: `0 <= n <= 10^4` nodes, `0 <= Node.val <= 10^4`, all values unique, `0 <= low <= high <= 10^4`.

Example: `[1,0,2], low = 1, high = 2` → `[1,null,2]`. Example: `[3,0,4,null,2,null,null,1], low = 1, high = 3` → `[3,2,null,1]`. Example: empty → empty.

hint: Use the BST property: if a node's value is below `low`, its entire left subtree is also below `low`, so nothing there can survive.
hint: When the current value is too small, the answer is the trimmed right subtree; when it is too large, the answer is the trimmed left subtree.
hint: If the value is in range, keep the node and recursively trim both of its children, reattaching the trimmed results.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
TreeNode* trimBST(TreeNode* root, int low, int high);
```

```cpp
TreeNode* trimBST(TreeNode* root, int low, int high) {
    if (!root) return nullptr;
    if (root->val < low)  return trimBST(root->right, low, high);
    if (root->val > high) return trimBST(root->left,  low, high);
    root->left  = trimBST(root->left,  low, high);
    root->right = trimBST(root->right, low, high);
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
static vector<optional<int>> serialize(TreeNode* root) {
    vector<optional<int>> out;
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
    using N = optional<int>;
    if (serialize(trimBST(build({1,0,2}), 1, 2)) != vector<optional<int>>({1,N{},2})) { std::puts("case1"); return 1; }
    if (serialize(trimBST(build({3,0,4,N{},2,N{},N{},1}), 1, 3)) !=
        vector<optional<int>>({3,2,N{},1})) { std::puts("case2"); return 1; }
    if (!serialize(trimBST(build({}), 0, 5)).empty()) { std::puts("case3"); return 1; }
    if (serialize(trimBST(build({8,3,10,1,6,N{},14,N{},N{},4,7,13}), 5, 13)) !=
        vector<optional<int>>({8,6,10,N{},7,N{},13})) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Recurse using the BST ordering. If a node's value is below `low`, everything in its left subtree is also too small, so the answer is the trimmed right subtree; symmetrically for values above `high`. In-range nodes are kept, with both children trimmed and reattached. Each node is examined at most once. O(n) time, O(h) space for recursion.
