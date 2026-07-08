## challenge: Dedupe Preserving Order
tags: dict, ordering
track: python
lang: python
difficulty: easy

Remove duplicate values from `lst`, keeping only the first occurrence of each and preserving the original relative order.

Constraints: elements are hashable; `lst` may be empty.

Example: `[1, 2, 2, 3, 1]` → `[1, 2, 3]`.

hint: A `set` removes duplicates but loses order — you need something ordered.
hint: Since Python 3.7, a plain `dict` preserves insertion order.
hint: `dict.fromkeys(lst)` builds a dict whose keys are the deduped items.

```python
# starter
def dedupe(lst):
    ...
```

```python
def dedupe(lst):
    return list(dict.fromkeys(lst))
```

```python
# harness
#__USER__
def _check():
    assert dedupe([1, 2, 2, 3, 1]) == [1, 2, 3]
    assert dedupe([]) == []
    assert dedupe([1]) == [1]
    assert dedupe([1, 1, 1]) == [1]
    assert dedupe(['a', 'b', 'a', 'c']) == ['a', 'b', 'c']
    print("PASS")

_check()
```

**Editorial:** `dict.fromkeys(lst)` uses the items as keys, so duplicates collapse automatically while insertion order is preserved (guaranteed since Python 3.7). Converting the keys back to a list gives an order-preserving dedupe in one line — unlike `set(lst)`, which is unordered, and unlike a manual `seen`-set loop.
