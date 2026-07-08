## challenge: Minimum Depth of Binary Tree
tags: tree, dfs, bfs
track: faang
difficulty: easy

Given the root of a binary tree, return its minimum depth: the number of nodes along the shortest path from the root down to the nearest leaf. A leaf is a node with no children.

Constraints: `0 <= n <= 10^5` nodes, `-1000 <= Node.val <= 1000`.

Example: `[3,9,20,null,null,15,7]` → `2`. Example: `[2,null,3,null,4,null,5,null,6]` → `5`. Example: empty → `0`.

hint: The minimum depth ends at a leaf; a node with only one child is not a leaf, so you cannot just take the smaller of the two subtree depths.
hint: If one child is missing, the path must continue through the child that exists — recurse only into that side.
hint: When both children are present, the answer is one plus the smaller of the two subtree minimum depths.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int minDepth(TreeNode* root);
```

```cpp
int minDepth(TreeNode* root) {
    if (!root) return 0;
    if (!root->left)  return 1 + minDepth(root->right);
    if (!root->right) return 1 + minDepth(root->left);
    return 1 + std::min(minDepth(root->left), minDepth(root->right));
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
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
    if (minDepth(build({3,9,20,N{},N{},15,7})) != 2)             { std::puts("case1"); return 1; }
    if (minDepth(build({2,N{},3,N{},4,N{},5,N{},6})) != 5)       { std::puts("case2"); return 1; }
    if (minDepth(build({})) != 0)                               { std::puts("case3"); return 1; }
    if (minDepth(build({1})) != 1)                              { std::puts("case4"); return 1; }
    if (minDepth(build({1,2})) != 2)                            { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A depth-first walk where the key subtlety is single-child nodes: they are not leaves, so the shortest path must go through the child that exists. Only when both children are present do we take one plus the smaller subtree depth; otherwise we recurse into the non-null side. Each node is visited once. O(n) time, O(h) space for recursion.
