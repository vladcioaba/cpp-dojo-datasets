## challenge: Path Sum
tags: tree, dfs, recursion
track: faang
difficulty: easy

Given the root of a binary tree and an integer `targetSum`, return `true` if the tree has a root-to-leaf path such that adding up all the values along the path equals `targetSum`. A leaf is a node with no children.

Constraints: `0 <= n <= 5000` nodes, `-1000 <= Node.val <= 1000`, `-10^9 <= targetSum <= 10^9`.

Example: `[5,4,8,11,null,13,4,7,2,null,null,null,1], targetSum = 22` → `true` (the path `5 -> 4 -> 11 -> 2`). Example: `[1,2,3], targetSum = 5` → `false`. Example: empty, `targetSum = 0` → `false`.

hint: The answer must end at a leaf — reaching the target partway down does not count unless that node is a leaf.
hint: Subtract the current node's value from the remaining target as you descend, then check the remainder at each leaf.
hint: At a leaf, succeed when the remaining target equals the leaf's value; otherwise recurse into whichever children exist and OR the results.

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
bool hasPathSum(TreeNode* root, int targetSum);
```

```cpp
bool hasPathSum(TreeNode* root, int targetSum) {
    if (!root) return false;
    if (!root->left && !root->right) return targetSum == root->val;
    int rem = targetSum - root->val;
    return hasPathSum(root->left, rem) || hasPathSum(root->right, rem);
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
    if (!hasPathSum(build({5,4,8,11,N{},13,4,7,2,N{},N{},N{},1}), 22)) { std::puts("case1"); return 1; }
    if ( hasPathSum(build({1,2,3}), 5))                                { std::puts("case2"); return 1; }
    if ( hasPathSum(build({}), 0))                                     { std::puts("case3"); return 1; }
    if ( hasPathSum(build({1,2}), 1))                                  { std::puts("case4"); return 1; }
    if (!hasPathSum(build({-2,N{},-3}), -5))                           { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A depth-first walk carries the remaining target downward. At every leaf, the path is valid exactly when the remaining amount equals the leaf's value; internal nodes report success if either subtree contains a qualifying path. Each node is visited once. O(n) time, O(h) space for recursion.
