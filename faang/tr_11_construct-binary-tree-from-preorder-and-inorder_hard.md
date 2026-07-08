## challenge: Construct Binary Tree from Preorder and Inorder Traversal
tags: tree, dfs, recursion
track: faang
difficulty: hard

Given two integer arrays `preorder` and `inorder` — the preorder and inorder traversals of the same binary tree with unique values — reconstruct and return the tree. The output is validated by comparing its level-order layout (`null` for missing children).

Constraints: `1 <= n <= 3000` nodes, values are unique and fit in `int`.

Example: `preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]` → `[3,9,20,null,null,15,7]`. Example: `preorder = [-1], inorder = [-1]` → `[-1]`. Example: `preorder = [1,2,3], inorder = [3,2,1]` → `[1,2,null,3]`.

hint: The first element of a preorder traversal is always the root of the (sub)tree.
hint: Locating that root inside the inorder array splits it into the left subtree's values and the right subtree's values.
hint: Consume preorder left-to-right with a moving index and recurse on the inorder sub-ranges; a hash map from value to inorder index makes the split O(1).

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder);
```

```cpp
TreeNode* buildTree(std::vector<int>& preorder, std::vector<int>& inorder) {
    std::unordered_map<int, int> idx;
    for (int i = 0; i < (int)inorder.size(); ++i) idx[inorder[i]] = i;
    int pre = 0;
    std::function<TreeNode*(int, int)> go = [&](int lo, int hi) -> TreeNode* {
        if (lo > hi) return nullptr;
        int v = preorder[pre++];
        TreeNode* node = new TreeNode(v);
        int m = idx[v];
        node->left  = go(lo, m - 1);
        node->right = go(m + 1, hi);
        return node;
    };
    return go(0, (int)inorder.size() - 1);
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <optional>
#include <unordered_map>
#include <functional>
using std::vector;
using std::optional;
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
static vector<optional<int>> toLevel(TreeNode* root) {
    vector<optional<int>> out;
    if (!root) return out;
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
    {
        vector<int> pre{3,9,20,15,7}, in{9,3,15,20,7};
        if (toLevel(buildTree(pre, in)) != vector<optional<int>>({3,9,20,N{},N{},15,7})) { std::puts("case1"); return 1; }
    }
    {
        vector<int> pre{-1}, in{-1};
        vector<optional<int>> want = {-1};
        if (toLevel(buildTree(pre, in)) != want) { std::puts("case2"); return 1; }
    }
    {
        vector<int> pre{1,2,3}, in{3,2,1};
        if (toLevel(buildTree(pre, in)) != vector<optional<int>>({1,2,N{},3})) { std::puts("case3"); return 1; }
    }
    {
        vector<int> pre{1,2,3}, in{1,2,3};
        if (toLevel(buildTree(pre, in)) != vector<optional<int>>({1,N{},2,N{},3})) { std::puts("case4"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The head of `preorder` is the current subtree's root; its position in `inorder` partitions the remaining values into the left and right subtrees. Recurse left then right, advancing a single shared index into `preorder`, so the ordering lines up automatically. A value-to-index map over `inorder` makes each split constant time. O(n) time, O(n) space.
