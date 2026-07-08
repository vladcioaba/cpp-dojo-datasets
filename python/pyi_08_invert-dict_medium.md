## challenge: Invert a Dict
tags: defaultdict, dict
track: python
lang: python
difficulty: medium

Invert a dictionary so each original value maps to the list of keys that had it. Because values need not be unique, every value maps to a list of keys.

Constraints: values are hashable; the dict may be empty. Key order within each list follows the iteration order of the input.

Example: `{'a': 1, 'b': 2, 'c': 1}` → `{1: ['a', 'c'], 2: ['b']}`.

hint: Each value becomes a key whose bucket collects one-or-more original keys.
hint: `collections.defaultdict(list)` gives you an empty list to append to on first touch.
hint: Iterate `for key, value in d.items()` and do `result[value].append(key)`.

```python
# starter
def invert(d):
    ...
```

```python
from collections import defaultdict

def invert(d):
    result = defaultdict(list)
    for key, value in d.items():
        result[value].append(key)
    return dict(result)
```

```python
# harness
#__USER__
def _norm(d):
    return {k: sorted(v) for k, v in d.items()}

def _check():
    assert _norm(invert({'a': 1, 'b': 2, 'c': 1})) == {1: ['a', 'c'], 2: ['b']}
    assert invert({}) == {}
    assert invert({'x': 5}) == {5: ['x']}
    assert _norm(invert({'a': 1, 'b': 1})) == {1: ['a', 'b']}
    assert _norm(invert({'a': 1, 'b': 2, 'c': 3})) == {1: ['a'], 2: ['b'], 3: ['c']}
    print("PASS")

_check()
```

**Editorial:** Because several keys can share a value, the inverse maps each value to a list. `defaultdict(list)` removes the "create the bucket if missing" branch, so the loop body is a single `result[value].append(key)`. Converting back with `dict(result)` returns an ordinary dict. This is the canonical one-to-many inversion idiom.
