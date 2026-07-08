## challenge: Merge Two Binary Trees
tags: tree, dfs, recursion
track: faang
difficulty: medium

Given the roots of two binary trees `root1` and `root2`, merge them into a new tree. Where both trees have a node in the same position, the merged node's value is the sum of the two; where only one tree has a node, that node (and its subtree) is used as-is. Return the root of the merged tree.

Constraints: `0 <= n <= 2000` nodes in each tree, `-10^4 <= Node.val <= 10^4`.

Example: `root1 = [1,3,2,5], root2 = [2,1,3,null,4,null,7]` → `[3,4,5,5,4,null,7]`. Example: `root1 = [1], root2 = [1,2]` → `[2,2]`. Example: both empty → empty.

hint: Recurse over both trees in lockstep, visiting the same position in each at the same time.
hint: If one side is null at a position, the answer there is simply the other side's whole subtree — no further work needed.
hint: When both sides are present, create a node holding the sum, then recurse into the two left children and the two right children.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2);
```

```cpp
TreeNode* mergeTrees(TreeNode* root1, TreeNode* root2) {
    if (!root1) return root2;
    if (!root2) return root1;
    TreeNode* node = new TreeNode(root1->val + root2->val);
    node->left  = mergeTrees(root1->left,  root2->left);
    node->right = mergeTrees(root1->right, root2->right);
    return node;
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
static vector<optional<int>> serialize(TreeNode* root) {
    vector<optional<int>> out;
    std::queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* c = q.front(); q.pop();
        if (c) { out.push_back(c->val); q.push(c->left); q.push(c->right); }
        else out.push_back(std::nullopt);
    }
    while (!out.empty() && !out.back().has_value()) out.pop_back();
    return out;
}
//__USER__
int main() {
    using N = optional<int>;
    if (serialize(mergeTrees(build({1,3,2,5}), build({2,1,3,N{},4,N{},7}))) !=
        vector<optional<int>>({3,4,5,5,4,N{},7})) { std::puts("case1"); return 1; }
    if (serialize(mergeTrees(build({1}), build({1,2}))) != vector<optional<int>>({2,2})) { std::puts("case2"); return 1; }
    if (!serialize(mergeTrees(build({}), build({}))).empty())                            { std::puts("case3"); return 1; }
    if (serialize(mergeTrees(build({}), build({5,6,7}))) != vector<optional<int>>({5,6,7})) { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Recurse through both trees simultaneously. Whenever one side is null, the other side's entire subtree is the answer for that position, so return it directly. When both nodes exist, allocate a node holding the sum and recurse into the paired left and right children. Work is proportional to the overlapping region. O(min(n1, n2)) time, O(min(h1, h2)) space for recursion.
