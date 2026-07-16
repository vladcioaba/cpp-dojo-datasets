## challenge: fix: Undo corrupts the board
tags: code-review, debugging, copying
track: python
lang: python
difficulty: medium

Code review found a bug: placing a mark on the "new" board also appears on the original board kept for undo, so undo restores a corrupted position. Find and fix it — keep the function signature.

hint: `list(board)` does make a new list — so what do the two lists contain?
hint: This is a shallow-vs-deep copy problem on a nested structure.
hint: The copied outer list still holds the *same* row objects; copy each row too (`[list(r) for r in board]`) or use `copy.deepcopy`.

```python
# starter
def with_move(board, row, col, mark):
    """Return a new board with `mark` placed at (row, col).

    The original board must be left untouched (it is kept for undo).
    """
    new_board = list(board)
    new_board[row][col] = mark
    return new_board
```

```python
def with_move(board, row, col, mark):
    """Return a new board with `mark` placed at (row, col).

    The original board must be left untouched (it is kept for undo).
    """
    new_board = [list(r) for r in board]
    new_board[row][col] = mark
    return new_board
```

```python
# harness
#__USER__
def _check():
    board = [[".", ".", "."], [".", ".", "."]]
    nb = with_move(board, 0, 2, "X")
    assert nb[0][2] == "X"
    assert board[0][2] == ".", "original board was modified"
    assert nb is not board
    nb2 = with_move(board, 1, 0, "O")
    assert nb2[1][0] == "O" and nb2[0][2] == "."
    assert board == [[".", ".", "."], [".", ".", "."]]
    print("PASS")

_check()
```

**Editorial:** `list(board)` copies only the outer list; its elements are references to the *same* row lists, so `new_board[row]` and `board[row]` are one object and writing a cell mutates both boards. The fix copies each level that will be mutated: `[list(r) for r in board]` (or `copy.deepcopy` for arbitrary nesting). Reviewers catch this by asking, whenever a copy of a nested structure is made, "shallow or deep — and which level gets mutated afterwards?"; `list(x)`, `x[:]`, `x.copy()`, and `copy.copy(x)` are all equally shallow.
