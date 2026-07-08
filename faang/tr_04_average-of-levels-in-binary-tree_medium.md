## challenge: Average of Levels in Binary Tree
tags: tree, bfs, queue
track: faang
difficulty: medium

Given the root of a binary tree, return an array of the average value of the nodes on each level, ordered from the root level down. Answers within `10^-5` of the true value are accepted.

Constraints: `1 <= n <= 10^4` nodes, `-2^31 <= Node.val <= 2^31 - 1`.

Example: `[3,9,20,null,null,15,7]` → `[3.0, 14.5, 11.0]`. Example: `[5,14,11,7,3,null,8]` → `[5.0, 12.5, 6.0]`. Example: `[1]` → `[1.0]`.

hint: You need the nodes grouped by depth, which is exactly what a level-by-level breadth-first traversal gives you.
hint: Snapshot the queue size at the start of each level so you know how many nodes belong to it before their children get enqueued.
hint: Accumulate the level's sum in a `double` (or `long long`) to avoid overflow, then divide by the level's node count.

```cpp
// starter
#include <vector>
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};
std::vector<double> averageOfLevels(TreeNode* root);
```

```cpp
std::vector<double> averageOfLevels(TreeNode* root) {
    std::vector<double> res;
    if (!root) return res;
    std::queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = (int)q.size();
        double sum = 0;
        for (int i = 0; i < sz; ++i) {
            TreeNode* c = q.front(); q.pop();
            sum += c->val;
            if (c->left)  q.push(c->left);
            if (c->right) q.push(c->right);
        }
        res.push_back(sum / sz);
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
#include <cmath>
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
static bool close(const vector<double>& r, const vector<double>& w) {
    if (r.size() != w.size()) return false;
    for (size_t i = 0; i < r.size(); ++i) if (std::fabs(r[i] - w[i]) > 1e-5) return false;
    return true;
}
//__USER__
int main() {
    if (!close(averageOfLevels(build({3,9,20,std::nullopt,std::nullopt,15,7})), {3.0,14.5,11.0})) { std::puts("case1"); return 1; }
    if (!close(averageOfLevels(build({3,9,20,15,7})), {3.0,14.5,11.0}))                            { std::puts("case2"); return 1; }
    if (!close(averageOfLevels(build({1})), {1.0}))                                                { std::puts("case3"); return 1; }
    if (!close(averageOfLevels(build({5,14,11,7,3,std::nullopt,8})), {5.0,12.5,6.0}))              { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Breadth-first traversal processes one level per outer iteration by fixing the level's node count from the queue size before enqueuing children. Summing the values in a wide integer or floating type sidesteps overflow, and dividing by the count yields each level's average. Every node is enqueued and dequeued once. O(n) time, O(n) space.
