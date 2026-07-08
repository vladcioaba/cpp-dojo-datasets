## challenge: Invert Binary Tree
tags: tree, dfs, recursion, binary-tree
track: python
lang: python
difficulty: easy

Given the `root` of a binary tree, invert the tree (swap every node's left and right child) and return its root.

Constraints: `0 <= number of nodes <= 100`, `-100 <= Node.val <= 100`.

Example: `root = [4, 2, 7, 1, 3, 6, 9]` → `[4, 7, 2, 9, 6, 3, 1]` (the tree is mirrored left-to-right).

hint: Inverting a tree means mirroring it: at every node, swap its two subtrees.
hint: Recurse first or swap first — either works, as long as every node is visited once.
hint: The base case is the empty subtree (`None`); return it unchanged.

```python
# starter
def invert_tree(root):
    ...
```

```python
def invert_tree(root):
    if root is None:
        return None
    root.left, root.right = invert_tree(root.right), invert_tree(root.left)
    return root
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

def _ser(root):
    out = []
    q = deque([root])
    while q:
        node = q.popleft()
        if node:
            out.append(node.val); q.append(node.left); q.append(node.right)
        else:
            out.append(None)
    while out and out[-1] is None:
        out.pop()
    return out

def _check():
    assert _ser(invert_tree(_build([4, 2, 7, 1, 3, 6, 9]))) == [4, 7, 2, 9, 6, 3, 1]
    assert _ser(invert_tree(_build([2, 1, 3]))) == [2, 3, 1]
    assert _ser(invert_tree(_build([]))) == []
    assert _ser(invert_tree(_build([1]))) == [1]
    print("PASS")

_check()
```

**Editorial:** Invert by swapping each node's children and recursing into both subtrees. The base case returns `None` for an empty subtree. Every node is touched once, so it is O(n) time and O(h) space for the recursion stack, where `h` is the tree height.
