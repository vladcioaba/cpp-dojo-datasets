## challenge: Binary Tree Level Order Traversal
tags: tree, bfs, queue
track: faang
difficulty: medium

Given the root of a binary tree, return its level-order traversal: a list of levels, each a list of node values from left to right, top level first.

Constraints: `0 <= n <= 2000` nodes, `-1000 <= Node.val <= 1000`.

Example: `[3,9,20,null,null,15,7]` → `[[3],[9,20],[15,7]]`. Example: `[1]` → `[[1]]`. Example: empty → `[]`.

hint: Process the tree one complete level at a time, from the top down.
hint: BFS with a queue, but snapshot the queue's size at the start of each level so you know where the level ends.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<std::vector<int>> levelOrder(TreeNode* root);
```

```cpp
std::vector<std::vector<int>> levelOrder(TreeNode* root) {
    std::vector<std::vector<int>> res;
    if (!root) return res;
    std::queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = (int)q.size();
        std::vector<int> level;
        for (int i = 0; i < sz; ++i) {
            TreeNode* c = q.front(); q.pop();
            level.push_back(c->val);
            if (c->left)  q.push(c->left);
            if (c->right) q.push(c->right);
        }
        res.push_back(std::move(level));
    }
    return res;
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
//__USER__
int main() {
    {
        auto r = levelOrder(build({3,9,20,std::nullopt,std::nullopt,15,7}));
        vector<vector<int>> want = {{3},{9,20},{15,7}};
        if (r != want) { std::puts("case1"); return 1; }
    }
    {
        auto r = levelOrder(build({1}));
        vector<vector<int>> want = {{1}};
        if (r != want) { std::puts("case2"); return 1; }
    }
    {
        auto r = levelOrder(build({}));
        if (!r.empty()) { std::puts("case3"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Breadth-first traversal with a queue, capturing the queue size at each level boundary to group the nodes per level. Every node is enqueued and dequeued once. O(n) time, O(n) space.
