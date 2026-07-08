## challenge: Insert into a Binary Search Tree
tags: tree, binary-search-tree, recursion
track: faang
difficulty: medium

Given the root of a binary search tree and a value `val` not already present, insert `val` into the tree so that it remains a valid BST, and return the root. Insertion at a leaf position is always possible; return any valid BST that results, though the canonical answer inserts at the natural leaf.

Constraints: `0 <= n <= 10^4` nodes, `-10^8 <= Node.val, val <= 10^8`, all values unique.

Example: `[4,2,7,1,3], val = 5` → `[4,2,7,1,3,5]`. Example: empty, `val = 5` → `[5]`. Example: `[40,20,60,10,30,50,70], val = 25` → inserts `25` as the right child of `10`... adjusting down to the correct leaf.

hint: The BST order tells you which way to go at each node: smaller values belong left, larger values belong right.
hint: Walk down following that rule until you reach a null spot — that empty position is exactly where the new value must live.
hint: The recursion returns the (possibly new) subtree root, so reattach the result to `root->left` or `root->right` and return `root`.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
TreeNode* insertIntoBST(TreeNode* root, int val);
```

```cpp
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);
    if (val < root->val) root->left  = insertIntoBST(root->left,  val);
    else                 root->right = insertIntoBST(root->right, val);
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
    if (serialize(insertIntoBST(build({4,2,7,1,3}), 5)) != vector<optional<int>>({4,2,7,1,3,5})) { std::puts("case1"); return 1; }
    if (serialize(insertIntoBST(build({}), 5)) != vector<optional<int>>({N{5}}))                 { std::puts("case2"); return 1; }
    if (serialize(insertIntoBST(build({40,20,60,10,30,50,70}), 25)) !=
        vector<optional<int>>({40,20,60,10,30,50,70,N{},N{},25}))                                { std::puts("case3"); return 1; }
    if (serialize(insertIntoBST(build({5,3,6,2,4,N{},7}), 1)) !=
        vector<optional<int>>({5,3,6,2,4,N{},7,1}))                                              { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Follow the BST ordering downward — go left when the new value is smaller, right when larger — until you fall off the tree at a null pointer. That empty slot is the correct home for the new node. Returning the subtree root from each call lets the parent reattach the (unchanged or newly created) child cleanly. The walk follows one root-to-leaf path. O(h) time, O(h) space for recursion.
