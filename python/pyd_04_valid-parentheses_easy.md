## challenge: Valid Parentheses
tags: stack, string
track: python
lang: python
difficulty: easy

Given a string `s` containing only the characters `()[]{}`, return `True` if every opening bracket is closed by the matching type in the correct order.

Constraints: `0 <= len(s) <= 10^4`, `s` consists only of the six bracket characters.

Example: `s = "()[]{}"` → `True`. `s = "([)]"` → `False` (closes `(` with `]`).

hint: The most recently opened bracket must be the first one closed — that is last-in-first-out.
hint: A stack of unmatched openers is the natural fit here.
hint: Push openers; on a closer, the top of the stack must be its matching opener or it is invalid.

```python
# starter
def is_valid(s):
    ...
```

```python
def is_valid(s):
    pairs = {')': '(', ']': '[', '}': '{'}
    stack = []
    for c in s:
        if c in pairs:
            if not stack or stack.pop() != pairs[c]:
                return False
        else:
            stack.append(c)
    return not stack
```

```python
# harness
#__USER__
def _check():
    assert is_valid("()") is True
    assert is_valid("()[]{}") is True
    assert is_valid("(]") is False
    assert is_valid("([)]") is False
    assert is_valid("{[]}") is True
    assert is_valid("") is True
    assert is_valid("(") is False
    print("PASS")

_check()
```

**Editorial:** Push every opening bracket onto a stack. When you hit a closing bracket, the correct match must be sitting on top of the stack — pop it and check it equals the expected opener; if the stack is empty or the top is wrong, the string is invalid. After scanning, a truly balanced string leaves the stack empty. O(n) time, O(n) space.
