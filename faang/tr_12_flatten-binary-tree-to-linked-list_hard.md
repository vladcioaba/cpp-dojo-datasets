## challenge: Flatten Binary Tree to Linked List
tags: tree, dfs, linked-list
track: faang
difficulty: hard

Given the root of a binary tree, flatten it in place into a "linked list": use the same `TreeNode` objects, where each node's `right` pointer points to the next node in pre-order traversal and each node's `left` pointer is set to `null`. The function returns nothing; it must mutate the tree.

Constraints: `0 <= n <= 2000` nodes, `-100 <= Node.val <= 100`. Aim for O(1) extra space beyond the recursion, if any.

Example: `[1,2,5,3,4,null,6]` → the tree becomes the right-skewed chain `1 -> 2 -> 3 -> 4 -> 5 -> 6`. Example: empty → empty. Example: `[0]` → `0`.

hint: The target order is exactly pre-order: node, then everything in the left subtree, then everything in the right subtree.
hint: For a node with a left subtree, the whole left subtree must be spliced in between the node and its current right subtree.
hint: Find the rightmost node of the left subtree, attach the original right subtree there, move the left subtree to the right, null out the left pointer, then advance.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
void flatten(TreeNode* root);
```

```cpp
void flatten(TreeNode* root) {
    TreeNode* cur = root;
    while (cur) {
        if (cur->left) {
            TreeNode* prev = cur->left;
            while (prev->right) prev = prev->right;
            prev->right = cur->right;
            cur->right = cur->left;
            cur->left = nullptr;
        }
        cur = cur->right;
    }
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
static bool allLeftNull(TreeNode* root) {
    for (TreeNode* c = root; c; c = c->right) if (c->left) return false;
    return true;
}
static vector<int> rightVals(TreeNode* root) {
    vector<int> out;
    for (TreeNode* c = root; c; c = c->right) out.push_back(c->val);
    return out;
}
//__USER__
int main() {
    using N = optional<int>;
    {
        TreeNode* t = build({1,2,5,3,4,N{},6});
        flatten(t);
        if (!allLeftNull(t) || rightVals(t) != vector<int>({1,2,3,4,5,6})) { std::puts("case1"); return 1; }
    }
    {
        TreeNode* t = build({});
        flatten(t);
        if (t != nullptr) { std::puts("case2"); return 1; }
    }
    {
        TreeNode* t = build({0});
        flatten(t);
        if (!allLeftNull(t) || rightVals(t) != vector<int>({0})) { std::puts("case3"); return 1; }
    }
    {
        TreeNode* t = build({1,2,3,4});
        flatten(t);
        if (!allLeftNull(t) || rightVals(t) != vector<int>({1,2,4,3})) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Walk the tree with a pointer. Whenever the current node has a left subtree, find that subtree's rightmost node, hang the current right subtree off it, then move the entire left subtree to the right side and clear the left pointer. Advancing along `right` visits nodes in pre-order and threads them into a single right-going chain, using no auxiliary data structure. O(n) time, O(1) extra space.
