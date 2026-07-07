## challenge: Validate Binary Search Tree
tags: tree, dfs, binary-search-tree
track: faang
difficulty: medium

Given the root of a binary tree, determine if it is a valid binary search tree: every node's left subtree contains only values strictly less than the node, its right subtree only values strictly greater, and both subtrees are themselves valid BSTs.

Constraints: `1 <= n <= 10^4` nodes, `-2^31 <= Node.val <= 2^31 - 1`.

Example: `[2,1,3]` → `true`. Example: `[5,1,4,null,null,3,6]` → `false` (3 and 6 are in the right subtree of 5 but 3 < 5). Example: `[5,4,6,null,null,3,7]` → `false`.

hint: Checking only against a node's immediate children is not enough — every node must fall within a range fixed by all of its ancestors.
hint: Recurse while carrying a valid open interval (lo, hi) that tightens as you descend left or right.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
bool isValidBST(TreeNode* root);
```

```cpp
static bool bstCheck(TreeNode* node, long long lo, long long hi) {
    if (!node) return true;
    if (node->val <= lo || node->val >= hi) return false;
    return bstCheck(node->left, lo, node->val) &&
           bstCheck(node->right, node->val, hi);
}
bool isValidBST(TreeNode* root) {
    return bstCheck(root, LLONG_MIN, LLONG_MAX);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <optional>
#include <climits>
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
    if (!isValidBST(build({2,1,3})))                           { std::puts("case1"); return 1; }
    if ( isValidBST(build({5,1,4,std::nullopt,std::nullopt,3,6}))) { std::puts("case2"); return 1; }
    if ( isValidBST(build({5,4,6,std::nullopt,std::nullopt,3,7}))) { std::puts("case3"); return 1; }
    if (!isValidBST(build({1})))                               { std::puts("case4"); return 1; }
    if ( isValidBST(build({2,2,2})))                           { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** DFS carries down an allowed open interval (lo, hi). Descending left tightens the upper bound to the current node's value; descending right tightens the lower bound. Any value outside its interval fails validation. O(n) time, O(h) space for recursion.
