## challenge: Validate Binary Search Tree
tags: tree, dfs, binary-search-tree
track: python
lang: python
difficulty: medium

Given the `root` of a binary tree, determine whether it is a valid binary search tree (BST). In a valid BST every node's value is strictly greater than all values in its left subtree and strictly less than all values in its right subtree.

Constraints: `0 <= number of nodes <= 10^4`, `-2^31 <= Node.val <= 2^31 - 1`.

Example: `root = [5, 1, 4, None, None, 3, 6]` → `False` (the node `3` sits in the right subtree of `5` but is smaller than `5`).

hint: It is not enough to compare a node only with its immediate children.
hint: Carry down an open interval `(lo, hi)` of allowed values for each node.
hint: A left child tightens the upper bound to the parent's value; a right child tightens the lower bound.

```python
# starter
def is_valid_bst(root):
    ...
```

```python
def is_valid_bst(root):
    def dfs(node, lo, hi):
        if node is None:
            return True
        if not (lo < node.val < hi):
            return False
        return dfs(node.left, lo, node.val) and dfs(node.right, node.val, hi)
    return dfs(root, float('-inf'), float('inf'))
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
    assert is_valid_bst(_build([2, 1, 3])) is True
    assert is_valid_bst(_build([5, 1, 4, None, None, 3, 6])) is False
    assert is_valid_bst(_build([])) is True
    assert is_valid_bst(_build([1])) is True
    assert is_valid_bst(_build([10, 5, 15, None, None, 6, 20])) is False
    assert is_valid_bst(_build([5, 3, 8, 1, 4, 7, 9])) is True
    print("PASS")

_check()
```

**Editorial:** Recurse with a valid open interval `(lo, hi)` that each node's value must fall inside. Going left shrinks `hi` to the parent value; going right raises `lo`. This catches violations that a naive parent-vs-child check misses (a deep descendant breaking an ancestor's bound). O(n) time, O(h) stack space.
