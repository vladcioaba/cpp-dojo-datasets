## challenge: Kth Smallest Element in a BST
tags: tree, dfs, binary-search-tree
track: faang
difficulty: hard

Given the root of a binary search tree and an integer `k`, return the value of the `k`th smallest element (1-indexed) among all node values in the tree.

Constraints: `1 <= k <= n <= 10^4` nodes, `0 <= Node.val <= 10^9`.

Example: `[3,1,4,null,2], k = 1` → `1`. Example: `[5,3,6,2,4,null,null,1], k = 3` → `3`. Example: `[1], k = 1` → `1`.

hint: An inorder traversal of a BST visits values in sorted ascending order.
hint: You do not need the whole sorted sequence — stop as soon as you have emitted `k` values.
hint: Use an iterative inorder walk with an explicit stack and a countdown of `k`; the node you land on when the counter hits zero is the answer.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int kthSmallest(TreeNode* root, int k);
```

```cpp
int kthSmallest(TreeNode* root, int k) {
    std::stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        if (--k == 0) return cur->val;
        cur = cur->right;
    }
    return -1;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <stack>
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
    if (kthSmallest(build({3,1,4,N{},2}), 1)       != 1) { std::puts("case1"); return 1; }
    if (kthSmallest(build({5,3,6,2,4,N{},N{},1}), 3) != 3) { std::puts("case2"); return 1; }
    if (kthSmallest(build({1}), 1)                 != 1) { std::puts("case3"); return 1; }
    if (kthSmallest(build({5,3,6,2,4,N{},N{},1}), 6) != 6) { std::puts("case4"); return 1; }
    if (kthSmallest(build({3,1,4,N{},2}), 4)       != 4) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Inorder traversal of a BST yields values in ascending order, so the kth node visited inorder is the kth smallest. An iterative stack-based inorder walk lets us stop early: decrement `k` each time a node is finalized and return when it reaches zero. In the worst case we visit O(h + k) nodes. O(h + k) time, O(h) stack space.
