## challenge: Binary Tree Level Order Traversal
tags: tree, bfs, binary-tree
track: python
lang: python
difficulty: medium

Given the `root` of a binary tree, return its level-order traversal: a list of levels, where each level is the list of node values at that depth (top to bottom).

Constraints: `0 <= number of nodes <= 2000`, `-1000 <= Node.val <= 1000`.

Example: `root = [3, 9, 20, None, None, 15, 7]` → `[[3], [9, 20], [15, 7]]`.

hint: Process the tree one depth at a time with a queue (breadth-first search).
hint: Before draining the queue, record its current length — that is the size of the current level.
hint: Pop exactly that many nodes, collect their values, and enqueue their children for the next level.

```python
# starter
def level_order(root):
    ...
```

```python
def level_order(root):
    from collections import deque
    if root is None:
        return []
    res = []
    q = deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        res.append(level)
    return res
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

def _canon(levels):
    return [sorted(l) for l in levels]

def _check():
    assert _canon(level_order(_build([3, 9, 20, None, None, 15, 7]))) == _canon([[3], [9, 20], [15, 7]])
    assert _canon(level_order(_build([1]))) == _canon([[1]])
    assert _canon(level_order(_build([]))) == _canon([])
    assert _canon(level_order(_build([1, 2, 3, 4, 5, 6, 7]))) == _canon([[1], [2, 3], [4, 5, 6, 7]])
    print("PASS")

_check()
```

**Editorial:** Breadth-first search with a queue. At the start of each outer iteration the queue holds exactly one level; snapshot its length, pop that many nodes into the current level list, and enqueue their children. O(n) time and O(n) space. The checker sorts within each level so any left/right ordering of equal-value siblings still validates.
