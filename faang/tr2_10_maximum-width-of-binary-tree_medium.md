## challenge: Maximum Width of Binary Tree
tags: tree, bfs, queue
track: faang
difficulty: medium

Given the root of a binary tree, return the maximum width among all levels. The width of a level is the number of positions between its leftmost and rightmost non-null nodes, counting the null gaps between them as if the tree were a complete binary tree at that level.

Constraints: `0 <= n <= 3000` nodes, `-100 <= Node.val <= 100`.

Example: `[1,3,2,5,3,null,9]` → `4` (the bottom level spans positions of `5` and `9` with two gaps between). Example: `[1,3,null,5,3]` → `2`. Example: `[1,3,2,5]` → `2`.

hint: Give each node the array index it would have in a complete binary tree: a node at index `i` has children at `2i` and `2i+1`.
hint: On each BFS level, the width is the last index minus the first index, plus one.
hint: Indices can grow huge on deep skewed trees; subtract the level's first index from every node so numbering restarts at zero each level and never overflows.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int widthOfBinaryTree(TreeNode* root);
```

```cpp
int widthOfBinaryTree(TreeNode* root) {
    if (!root) return 0;
    unsigned long long best = 0;
    std::queue<std::pair<TreeNode*, unsigned long long>> q;
    q.push({root, 0});
    while (!q.empty()) {
        int sz = (int)q.size();
        unsigned long long first = q.front().second;
        unsigned long long last = first;
        for (int i = 0; i < sz; ++i) {
            auto [node, idx] = q.front(); q.pop();
            unsigned long long norm = idx - first;
            last = norm;
            if (node->left)  q.push({node->left,  norm * 2});
            if (node->right) q.push({node->right, norm * 2 + 1});
        }
        best = std::max(best, last + 1);
    }
    return (int)best;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <utility>
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
    if (widthOfBinaryTree(build({1,3,2,5,3,N{},9})) != 4) { std::puts("case1"); return 1; }
    if (widthOfBinaryTree(build({1,3,N{},5,3})) != 2)     { std::puts("case2"); return 1; }
    if (widthOfBinaryTree(build({1,3,2,5})) != 2)         { std::puts("case3"); return 1; }
    if (widthOfBinaryTree(build({1})) != 1)               { std::puts("case4"); return 1; }
    if (widthOfBinaryTree(build({1,1,1,1,N{},N{},1,1,N{},N{},1})) != 8) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Run BFS while tagging each node with the index it would occupy in a complete binary tree — left child `2i`, right child `2i+1`. A level's width is its last minus first index plus one. To keep the indices from overflowing on deep, sparse trees, renumber each level relative to its first node's index so counting restarts at zero. Each node is processed once. O(n) time, O(n) space.
