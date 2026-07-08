## challenge: Maximum Depth of Binary Tree
tags: tree, dfs, bfs
track: faang
difficulty: easy

Given the root of a binary tree, return its maximum depth: the number of nodes along the longest path from the root node down to the farthest leaf node.

Constraints: `0 <= n <= 10^4` nodes, `-100 <= Node.val <= 100`.

Example: `[3,9,20,null,null,15,7]` -> `3`. Example: empty -> `0`. Example: `[1]` -> `1`.

hint: The depth of a tree is one more than the depth of its deeper subtree.
hint: A post-order DFS recursion is the cleanest tool here; a BFS level count works too.
hint: Recurse into both children and return `1 + max(leftDepth, rightDepth)`; an empty subtree contributes depth `0`.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int maxDepth(TreeNode* root);
```

```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;
    return 1 + std::max(maxDepth(root->left), maxDepth(root->right));
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <optional>
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
    if (maxDepth(build({3,9,20,std::nullopt,std::nullopt,15,7})) != 3) { std::puts("case1"); return 1; }
    if (maxDepth(build({})) != 0) { std::puts("case2"); return 1; }
    if (maxDepth(build({1})) != 1) { std::puts("case3"); return 1; }
    if (maxDepth(build({1,2,std::nullopt,3})) != 3) { std::puts("case4"); return 1; }  // left-skewed 1->2->3
    std::puts("PASS");
}
```

**Editorial:** Post-order recursion: the depth of any node is one plus the maximum depth of its two children, with the null base case returning zero. Every node is visited once. O(n) time, O(h) space for the recursion stack where h is the tree height.
