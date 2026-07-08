## challenge: Path Sum III
tags: tree, dfs, prefix-sum
track: faang
difficulty: hard

Given the root of a binary tree and an integer `targetSum`, return the number of downward paths whose node values sum to `targetSum`. A path does not need to start at the root or end at a leaf, but it must go strictly downward (from a node to one of its descendants, following parent-to-child links).

Constraints: `0 <= n <= 1000` nodes, `-10^9 <= Node.val <= 10^9`, `-1000 <= targetSum <= 1000`. Path sums can exceed 32-bit range, so accumulate with 64-bit integers.

Example: `[10,5,-3,3,2,null,11,3,-2,null,1], targetSum = 8` → `3`. Example: `[5,4,8,11,null,13,4,7,2,null,null,5,1], targetSum = 22` → `3`. Example: empty, `targetSum = 0` → `0`.

hint: A brute force that restarts a sum from every node is O(n^2); the prefix-sum trick brings it to O(n).
hint: Keep the running sum from the root to the current node, and a map counting how many times each prefix sum has occurred on the current root-to-node path.
hint: A downward path ending at the current node sums to `target` whenever some earlier prefix equals `current - target`; add the count of those, then remember to decrement the map entry as you unwind (backtrack).

```cpp
// starter
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
int pathSum(TreeNode* root, int targetSum);
```

```cpp
int pathSum(TreeNode* root, int targetSum) {
    std::unordered_map<long long, int> prefix;
    prefix[0] = 1;
    int count = 0;
    std::function<void(TreeNode*, long long)> dfs = [&](TreeNode* node, long long cur) {
        if (!node) return;
        cur += node->val;
        auto it = prefix.find(cur - targetSum);
        if (it != prefix.end()) count += it->second;
        ++prefix[cur];
        dfs(node->left,  cur);
        dfs(node->right, cur);
        --prefix[cur];
    };
    dfs(root, 0);
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <unordered_map>
#include <functional>
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
    if (pathSum(build({10,5,-3,3,2,N{},11,3,-2,N{},1}), 8) != 3)       { std::puts("case1"); return 1; }
    if (pathSum(build({5,4,8,11,N{},13,4,7,2,N{},N{},5,1}), 22) != 3)  { std::puts("case2"); return 1; }
    if (pathSum(build({}), 0) != 0)                                    { std::puts("case3"); return 1; }
    if (pathSum(build({1}), 1) != 1)                                   { std::puts("case4"); return 1; }
    if (pathSum(build({0,1,1}), 1) != 4)                               { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Turn "sum of a contiguous downward segment equals target" into a prefix-sum problem. Maintain the running sum from the root to the current node and a hash map counting each prefix sum seen along the current path. A qualifying path ending here exists once for every earlier prefix equal to `current - target`, so add those counts. Insert the current prefix before recursing and remove it afterward so only ancestors of the current node remain in the map. Using 64-bit sums avoids overflow. O(n) time, O(n) space.
