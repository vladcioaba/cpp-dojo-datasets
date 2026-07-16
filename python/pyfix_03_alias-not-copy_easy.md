## challenge: fix: Defaults drift after every request
tags: code-review, debugging, aliasing
track: python
lang: python
difficulty: easy

Code review found a bug: after one request passes custom settings, every later request mysteriously sees those custom values baked into the shared defaults. Find and fix it — keep the function signature.

hint: The docstring promises not to modify either argument — check whether that's true.
hint: Assignment in Python never copies data; it only creates another name for the same object.
hint: `settings = defaults` aliases the caller's dict, so `settings.update(...)` mutates the original — make a copy first.

```python
# starter
def merge_settings(defaults, overrides):
    """Combine `defaults` with per-request `overrides` (overrides win).

    Must not modify either argument.
    """
    settings = defaults
    settings.update(overrides)
    return settings
```

```python
def merge_settings(defaults, overrides):
    """Combine `defaults` with per-request `overrides` (overrides win).

    Must not modify either argument.
    """
    settings = dict(defaults)
    settings.update(overrides)
    return settings
```

```python
# harness
#__USER__
def _check():
    defaults = {"theme": "light", "retries": 3}
    out = merge_settings(defaults, {"theme": "dark"})
    assert out == {"theme": "dark", "retries": 3}
    assert defaults == {"theme": "light", "retries": 3}, "defaults were mutated"
    out2 = merge_settings(defaults, {})
    assert out2 == defaults and out2 is not defaults
    print("PASS")

_check()
```

**Editorial:** `settings = defaults` does not copy anything — both names point at the same dict, so `settings.update(overrides)` writes the per-request overrides straight into the shared defaults, poisoning every subsequent call. The fix is to copy first: `settings = dict(defaults)` (or `defaults.copy()`, or `{**defaults, **overrides}`). Reviewers spot this by treating every `a = b` on a mutable value followed by mutation as a red flag, especially when a docstring or contract promises the inputs stay untouched.
