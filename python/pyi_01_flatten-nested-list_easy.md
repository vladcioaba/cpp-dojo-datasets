## challenge: Flatten a Nested List
tags: comprehension, nested-list
track: python
lang: python
difficulty: easy

Given a list of lists `nested`, return a single flat list containing every element, flattening exactly one level of nesting (left to right).

Constraints: elements may be of any type; sublists may be empty; the input may be empty.

Example: `nested = [[1, 2], [3], [4, 5]]` → `[1, 2, 3, 4, 5]`.

hint: A list comprehension can carry two `for` clauses — an outer one and an inner one.
hint: The clause order reads like nested loops: `for sub in nested` then `for x in sub`.
hint: `[x for sub in nested for x in sub]` — no manual `append`, no `extend`.

```python
# starter
def flatten(nested):
    ...
```

```python
def flatten(nested):
    return [x for sub in nested for x in sub]
```

```python
# harness
#__USER__
def _check():
    assert flatten([[1, 2], [3], [4, 5]]) == [1, 2, 3, 4, 5]
    assert flatten([]) == []
    assert flatten([[]]) == []
    assert flatten([[7]]) == [7]
    assert flatten([[], [1], [2, 3]]) == [1, 2, 3]
    assert flatten([[1, 1], [1]]) == [1, 1, 1]
    print("PASS")

_check()
```

**Editorial:** A nested list comprehension with two `for` clauses flattens one level in a single expression. The clauses read top-to-bottom exactly like nested `for` loops (`for sub in nested` outer, `for x in sub` inner), but build the result list directly — no accumulator variable and no `.append`/`.extend` bookkeeping to get wrong.
