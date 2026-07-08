## challenge: Binary Tree Inorder Traversal
tags: tree, dfs, stack
track: faang
difficulty: easy

Given the root of a binary tree, return the inorder traversal of its nodes' values: for every node, visit its left subtree, then the node, then its right subtree.

Constraints: `0 <= n <= 100` nodes, `-100 <= Node.val <= 100`.

Example: `[1,null,2,3]` → `[1,3,2]`. Example: `[1]` → `[1]`. Example: empty → `[]`.

hint: Inorder means the recursion order left, node, right; a straight recursive walk appending to a list is the simplest correct answer.
hint: To do it iteratively, use an explicit stack: keep pushing left children until you run out, then pop, record, and move right.
hint: The loop condition is "while the current pointer is non-null OR the stack is non-empty" — that keeps you going until every node is processed.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<int> inorderTraversal(TreeNode* root);
```

```cpp
std::vector<int> inorderTraversal(TreeNode* root) {
    std::vector<int> res;
    std::stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        res.push_back(cur->val);
        cur = cur->right;
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
    if (inorderTraversal(build({1,std::nullopt,2,3})) != vector<int>({1,3,2})) { std::puts("case1"); return 1; }
    if (inorderTraversal(build({1})) != vector<int>({1})) { std::puts("case2"); return 1; }
    if (!inorderTraversal(build({})).empty()) { std::puts("case3"); return 1; }
    if (inorderTraversal(build({1,2,3,4,5,6,7})) != vector<int>({4,2,5,1,6,3,7})) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Simulate the recursion with an explicit stack. Descend left, pushing each node; when there is no left child, pop the top, record its value, and move to its right subtree to repeat. Each node is pushed and popped once. O(n) time, O(h) space for the stack where h is the tree height.
