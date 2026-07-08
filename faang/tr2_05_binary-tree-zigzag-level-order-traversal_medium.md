## challenge: Binary Tree Zigzag Level Order Traversal
tags: tree, bfs, queue
track: faang
difficulty: medium

Given the root of a binary tree, return the zigzag level-order traversal of its nodes' values: a list of levels top to bottom, where the first level is read left to right, the next right to left, and so on alternating.

Constraints: `0 <= n <= 2000` nodes, `-100 <= Node.val <= 100`.

Example: `[3,9,20,null,null,15,7]` → `[[3],[20,9],[15,7]]`. Example: `[1]` → `[[1]]`. Example: empty → `[]`.

hint: This is ordinary level-order BFS; the zigzag is only about how you lay each level's values into the output list.
hint: Keep visiting children strictly left to right so the tree structure stays simple; flip only the placement, not the traversal.
hint: Track a direction flag per level: on a right-to-left level, write each dequeued value into the level slot from the back instead of the front.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<std::vector<int>> zigzagLevelOrder(TreeNode* root);
```

```cpp
std::vector<std::vector<int>> zigzagLevelOrder(TreeNode* root) {
    std::vector<std::vector<int>> res;
    if (!root) return res;
    std::queue<TreeNode*> q; q.push(root);
    bool leftToRight = true;
    while (!q.empty()) {
        int sz = (int)q.size();
        std::vector<int> level(sz);
        for (int i = 0; i < sz; ++i) {
            TreeNode* cur = q.front(); q.pop();
            int idx = leftToRight ? i : sz - 1 - i;
            level[idx] = cur->val;
            if (cur->left)  q.push(cur->left);
            if (cur->right) q.push(cur->right);
        }
        res.push_back(std::move(level));
        leftToRight = !leftToRight;
    }
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <utility>
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
    {
        auto r = zigzagLevelOrder(build({3,9,20,N{},N{},15,7}));
        vector<vector<int>> want = {{3},{20,9},{15,7}};
        if (r != want) { std::puts("case1"); return 1; }
    }
    {
        auto r = zigzagLevelOrder(build({1}));
        vector<vector<int>> want = {{1}};
        if (r != want) { std::puts("case2"); return 1; }
    }
    {
        if (!zigzagLevelOrder(build({})).empty()) { std::puts("case3"); return 1; }
    }
    {
        auto r = zigzagLevelOrder(build({1,2,3,4,5,6,7}));
        vector<vector<int>> want = {{1},{3,2},{4,5,6,7}};
        if (r != want) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Run a normal breadth-first traversal, always enqueuing children left to right, and use a boolean that flips each level. Pre-size each level's vector and, on right-to-left levels, place the i-th dequeued value at position `sz - 1 - i`. This avoids reversing and keeps every node enqueued and dequeued once. O(n) time, O(n) space.
