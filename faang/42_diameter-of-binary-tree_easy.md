## challenge: Diameter of Binary Tree
tags: tree, dfs
track: faang
difficulty: easy

Given the root of a binary tree, return the length of its diameter: the number of edges on the longest path between any two nodes in the tree. This path may or may not pass through the root.

Constraints: `1 <= n <= 10^4` nodes, `-100 <= Node.val <= 100`.

Example: `[1,2,3,4,5]` -> `3` (the path `4 -> 2 -> 1 -> 3` or `5 -> 2 -> 1 -> 3` has 3 edges). Example: `[1,2]` -> `1`. Example: `[1]` -> `0`.

hint: The longest path through a given node uses the node's two deepest reaching branches, and its edge count equals leftHeight + rightHeight.
hint: Do a single post-order pass that returns each subtree's height while separately tracking the best leftHeight + rightHeight seen at any node.
hint: Define an empty subtree's height as `0`; a node's returned height is `1 + max(childHeights)`, and you update the answer before returning.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int diameterOfBinaryTree(TreeNode* root);
```

```cpp
int diameterOfBinaryTree(TreeNode* root) {
    int best = 0;
    std::function<int(TreeNode*)> height = [&](TreeNode* node) -> int {
        if (!node) return 0;
        int l = height(node->left);
        int r = height(node->right);
        best = std::max(best, l + r);
        return 1 + std::max(l, r);
    };
    height(root);
    return best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <optional>
#include <functional>
#include <algorithm>
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
    if (diameterOfBinaryTree(build({1,2,3,4,5})) != 3) { std::puts("case1"); return 1; }
    if (diameterOfBinaryTree(build({1,2})) != 1) { std::puts("case2"); return 1; }
    if (diameterOfBinaryTree(build({1})) != 0) { std::puts("case3"); return 1; }
    if (diameterOfBinaryTree(build({})) != 0) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A single post-order traversal returns each subtree's height while updating a running best of leftHeight + rightHeight, which is exactly the edge count of the longest path bending at that node. Every node is visited once. O(n) time, O(h) recursion depth.
