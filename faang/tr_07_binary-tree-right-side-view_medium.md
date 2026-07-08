## challenge: Binary Tree Right Side View
tags: tree, bfs, queue
track: faang
difficulty: medium

Given the root of a binary tree, imagine standing on its right side. Return the values of the nodes you can see, ordered from top to bottom — that is, the rightmost node of each level.

Constraints: `0 <= n <= 100` nodes, `-100 <= Node.val <= 100`.

Example: `[1,2,3,null,5,null,4]` → `[1,3,4]`. Example: `[1,null,3]` → `[1,3]`. Example: empty → `[]`.

hint: "What you see from the right" is just the last node encountered on each level.
hint: A level-order BFS lets you identify the final node of every level.
hint: Track each level's node count from the queue size, and record the value when you reach the last node of that level.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<int> rightSideView(TreeNode* root);
```

```cpp
std::vector<int> rightSideView(TreeNode* root) {
    std::vector<int> res;
    if (!root) return res;
    std::queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = (int)q.size();
        for (int i = 0; i < sz; ++i) {
            TreeNode* c = q.front(); q.pop();
            if (i == sz - 1) res.push_back(c->val);
            if (c->left)  q.push(c->left);
            if (c->right) q.push(c->right);
        }
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
    using N = optional<int>;
    if (rightSideView(build({1,2,3,N{},5,N{},4})) != vector<int>({1,3,4})) { std::puts("case1"); return 1; }
    if (rightSideView(build({1,N{},3}))           != vector<int>({1,3}))   { std::puts("case2"); return 1; }
    if (!rightSideView(build({})).empty())                                 { std::puts("case3"); return 1; }
    if (rightSideView(build({1,2,3,4}))           != vector<int>({1,3,4})) { std::puts("case4"); return 1; }
    if (rightSideView(build({1,2,N{},4}))         != vector<int>({1,2,4})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Run a level-order breadth-first traversal and, using the queue size to bound each level, emit the value of the last node dequeued on that level. This captures exactly the rightmost node visible per level, whether it is a left or right child. Every node is enqueued and dequeued once. O(n) time, O(n) space.
