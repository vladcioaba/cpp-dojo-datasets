## challenge: Same Tree
tags: tree, dfs
track: faang
difficulty: easy

Given the roots of two binary trees `p` and `q`, return `true` if they are structurally identical and every corresponding pair of nodes has the same value, otherwise return `false`.

Constraints: `0 <= n <= 100` nodes in each tree, `-10^4 <= Node.val <= 10^4`.

Example: `p = [1,2,3], q = [1,2,3]` -> `true`. Example: `p = [1,2], q = [1,null,2]` -> `false` (same values, different shape). Example: `p = [1,2,1], q = [1,1,2]` -> `false`. Example: both empty -> `true`.

hint: Two trees match only if their roots match and, recursively, their left subtrees and their right subtrees each match.
hint: A single lockstep DFS over both trees at once is all you need.
hint: Compare the two nodes in lockstep: both null is a match, exactly one null is a mismatch, otherwise compare values then recurse on both children.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
bool isSameTree(TreeNode* p, TreeNode* q);
```

```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
    if (!p && !q) return true;
    if (!p || !q) return false;
    if (p->val != q->val) return false;
    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
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
    if (!isSameTree(build({1,2,3}), build({1,2,3}))) { std::puts("case1"); return 1; }
    if (isSameTree(build({1,2}), build({1,std::nullopt,2}))) { std::puts("case2"); return 1; }
    if (isSameTree(build({1,2,1}), build({1,1,2}))) { std::puts("case3"); return 1; }
    if (!isSameTree(build({}), build({}))) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Walk both trees together with a single recursion: two nulls agree, one null disagrees, otherwise the values must be equal and both subtree pairs must match. Each node pair is compared once. O(min(np, nq)) time, O(min(hp, hq)) recursion depth.
