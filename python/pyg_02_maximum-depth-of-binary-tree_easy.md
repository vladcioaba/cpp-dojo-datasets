## challenge: Maximum Depth of Binary Tree
tags: tree, dfs, bfs, binary-tree
track: python
lang: python
difficulty: easy

Given the `root` of a binary tree, return its maximum depth — the number of nodes along the longest path from the root down to the farthest leaf.

Constraints: `0 <= number of nodes <= 10^4`, `-100 <= Node.val <= 100`.

Example: `root = [3, 9, 20, None, None, 15, 7]` → `3` (the path `3 → 20 → 15` has three nodes).

hint: The depth of a tree is 1 plus the depth of its deeper subtree.
hint: An empty tree has depth 0 — that is your base case.
hint: `depth(node) = 1 + max(depth(left), depth(right))`.

```python
# starter
def max_depth(root):
    ...
```

```python
def max_depth(root):
    if root is None:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

```python
# harness
#__USER__
from collections import deque

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def _build(vals):
    if not vals:
        return None
    root = TreeNode(vals[0])
    q = deque([root])
    i = 1
    while q and i < len(vals):
        node = q.popleft()
        if i < len(vals) and vals[i] is not None:
            node.left = TreeNode(vals[i]); q.append(node.left)
        i += 1
        if i < len(vals) and vals[i] is not None:
            node.right = TreeNode(vals[i]); q.append(node.right)
        i += 1
    return root

def _check():
    assert max_depth(_build([3, 9, 20, None, None, 15, 7])) == 3
    assert max_depth(_build([1, None, 2])) == 2
    assert max_depth(_build([])) == 0
    assert max_depth(_build([1])) == 1
    assert max_depth(_build([1, 2, 3, 4, None, None, 5])) == 3
    print("PASS")

_check()
```

**Editorial:** A post-order recursion: the depth at any node is one more than the maximum depth of its two children, with the empty subtree contributing 0. Each node is visited once for O(n) time and O(h) recursion-stack space.
