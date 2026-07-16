## challenge: fix: Blank lines survive the cleaner
tags: code-review, debugging, strings
track: python
lang: python
difficulty: easy

Code review found a bug: the log cleaner still emits lines of pure whitespace, and the lines it keeps still carry their trailing newlines. Find and fix it — keep the function signature.

hint: Watch what happens to each `line` inside the loop, statement by statement.
hint: Strings are immutable — string methods never modify in place.
hint: `line.strip()` computes a new string and returns it; if you don't bind the result, it's discarded.

```python
# starter
def clean_lines(lines):
    """Trim whitespace from each line; drop lines that are empty after trimming."""
    cleaned = []
    for line in lines:
        line.strip()
        if line:
            cleaned.append(line)
    return cleaned
```

```python
def clean_lines(lines):
    """Trim whitespace from each line; drop lines that are empty after trimming."""
    cleaned = []
    for line in lines:
        line = line.strip()
        if line:
            cleaned.append(line)
    return cleaned
```

```python
# harness
#__USER__
def _check():
    assert clean_lines(["  alpha  ", "", "   ", "beta\n"]) == ["alpha", "beta"]
    assert clean_lines(["\t\n", " "]) == []
    assert clean_lines([]) == []
    assert clean_lines(["ok"]) == ["ok"]
    print("PASS")

_check()
```

**Editorial:** Strings are immutable, so `line.strip()` cannot change `line` — it builds a new string and returns it, and here the return value was thrown away. The untouched `line` is then tested (a whitespace-only string is truthy) and appended verbatim. The fix is to rebind: `line = line.strip()`. Reviewers catch this by flagging any bare string-method call used as a statement — `s.strip()`, `s.replace(...)`, `s.upper()` on their own line are almost always bugs.
