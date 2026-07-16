## challenge: fix: Dedupe remembers too much
tags: code-review, debugging, mutable-default
track: python
lang: python
difficulty: medium

Code review found a bug: the first call works, but later calls silently drop items that were never in their own input — as if the function remembers previous batches. Find and fix it — keep the function signature.

hint: Each call in isolation is correct; the bug only appears across multiple calls.
hint: When is a default parameter value evaluated — per call, or once?
hint: `seen=set()` is created once at `def` time and shared by every call that omits it; use a `None` sentinel and create the set inside.

```python
# starter
def dedupe(items, seen=set()):
    """Return `items` without duplicates, preserving first-seen order."""
    out = []
    for x in items:
        if x not in seen:
            seen.add(x)
            out.append(x)
    return out
```

```python
def dedupe(items, seen=None):
    """Return `items` without duplicates, preserving first-seen order."""
    if seen is None:
        seen = set()
    out = []
    for x in items:
        if x not in seen:
            seen.add(x)
            out.append(x)
    return out
```

```python
# harness
#__USER__
def _check():
    assert dedupe([1, 2, 1, 3]) == [1, 2, 3]
    assert dedupe([3, 4, 4, 5]) == [3, 4, 5], "state leaked from a previous call"
    assert dedupe(["a", "a"]) == ["a"]
    assert dedupe([1, 1]) == [1], "state leaked from a previous call"
    print("PASS")

_check()
```

**Editorial:** Default values are evaluated exactly once, when the `def` statement runs — so the single `set()` object is shared by every call that doesn't pass `seen`, and each call inherits everything previous calls added to it. The second call drops `3` because the first call already "saw" it. The canonical fix is the `None` sentinel: default to `None` and build a fresh `set()` inside the body. Reviewers flag any mutable default (`[]`, `{}`, `set()`) on sight; the tell in the wild is call-order-dependent behavior.
