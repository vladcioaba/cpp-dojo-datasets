## challenge: Binary Tree Postorder Traversal
tags: tree, dfs, stack
track: faang
difficulty: easy

Given the root of a binary tree, return the postorder traversal of its nodes' values: for every node, visit its left subtree, then its right subtree, then the node itself.

Constraints: `0 <= n <= 100` nodes, `-100 <= Node.val <= 100`.

Example: `[1,null,2,3]` → `[3,2,1]`. Example: `[1]` → `[1]`. Example: empty → `[]`.

hint: Postorder is node last; a plain recursive walk that appends the value after recursing into both subtrees is the simplest correct answer.
hint: A neat iterative trick: produce the order node, right, left with a stack, then reverse the result.
hint: Node-right-left reversed is exactly left-right-node, which is postorder — so push the left child before the right child and reverse at the end.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<int> postorderTraversal(TreeNode* root);
```

```cpp
std::vector<int> postorderTraversal(TreeNode* root) {
    std::vector<int> res;
    if (!root) return res;
    std::stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* cur = st.top(); st.pop();
        res.push_back(cur->val);
        if (cur->left)  st.push(cur->left);
        if (cur->right) st.push(cur->right);
    }
    std::reverse(res.begin(), res.end());
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <stack>
#include <algorithm>
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
    if (postorderTraversal(build({1,N{},2,3})) != vector<int>({3,2,1})) { std::puts("case1"); return 1; }
    if (postorderTraversal(build({1})) != vector<int>({1}))             { std::puts("case2"); return 1; }
    if (!postorderTraversal(build({})).empty())                        { std::puts("case3"); return 1; }
    if (postorderTraversal(build({1,2,3,4,5})) != vector<int>({4,5,2,3,1})) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Rather than tracking whether both children are done, generate the easier order node, right, left with a single stack (push left before right), then reverse. The reverse of node-right-left is left-right-node, exactly postorder. Each node is pushed and popped once. O(n) time, O(n) space.
