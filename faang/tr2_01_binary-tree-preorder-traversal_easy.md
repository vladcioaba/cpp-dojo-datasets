## challenge: Binary Tree Preorder Traversal
tags: tree, dfs, stack
track: faang
difficulty: easy

Given the root of a binary tree, return the preorder traversal of its nodes' values: for every node, visit the node itself first, then its left subtree, then its right subtree.

Constraints: `0 <= n <= 100` nodes, `-100 <= Node.val <= 100`.

Example: `[1,null,2,3]` → `[1,2,3]`. Example: `[1]` → `[1]`. Example: empty → `[]`.

hint: Preorder means node, then left, then right; a plain recursive walk that appends the value before recursing is the simplest correct answer.
hint: To do it iteratively, push the root on a stack, then repeatedly pop a node, record it, and push its children.
hint: Push the right child before the left child so that the left child comes off the top of the stack first.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<int> preorderTraversal(TreeNode* root);
```

```cpp
std::vector<int> preorderTraversal(TreeNode* root) {
    std::vector<int> res;
    if (!root) return res;
    std::stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* cur = st.top(); st.pop();
        res.push_back(cur->val);
        if (cur->right) st.push(cur->right);
        if (cur->left)  st.push(cur->left);
    }
    return res;
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
    if (preorderTraversal(build({1,N{},2,3})) != vector<int>({1,2,3})) { std::puts("case1"); return 1; }
    if (preorderTraversal(build({1})) != vector<int>({1}))            { std::puts("case2"); return 1; }
    if (!preorderTraversal(build({})).empty())                       { std::puts("case3"); return 1; }
    if (preorderTraversal(build({1,2,3,4,5})) != vector<int>({1,2,4,5,3})) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Simulate the recursion with an explicit stack. Pop a node, record its value, then push its right child followed by its left child so the left is processed next — reproducing the node, left, right order. Each node is pushed and popped once. O(n) time, O(h) space for the stack where h is the tree height.
