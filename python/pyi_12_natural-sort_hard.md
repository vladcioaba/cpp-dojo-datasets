## challenge: Natural Sort Order
tags: sorting-key, regex
track: python
lang: python
difficulty: hard

Sort a list of strings in *natural* order, where embedded runs of digits compare by numeric value rather than lexicographically — so `"file2"` sorts before `"file10"` (a plain string sort would put `"file10"` first).

Constraints: strings mix letters and digit runs; the list may be empty.

Example: `["file10", "file2", "file1"]` → `["file1", "file2", "file10"]`.

hint: Split each string into alternating text and number chunks: `re.split(r'(\d+)', s)`.
hint: Convert the digit chunks to `int` so they compare numerically, leaving text chunks as strings.
hint: Use that mixed list as the `key`: `sorted` then compares the lists element-by-element.

```python
# starter
def natural_sort(strings):
    ...
```

```python
import re

def natural_sort(strings):
    def key(s):
        return [int(part) if part.isdigit() else part
                for part in re.split(r'(\d+)', s)]
    return sorted(strings, key=key)
```

```python
# harness
#__USER__
def _check():
    assert natural_sort(["file10", "file2", "file1"]) == ["file1", "file2", "file10"]
    assert natural_sort([]) == []
    assert natural_sort(["a"]) == ["a"]
    assert natural_sort(["img12", "img2", "img1"]) == ["img1", "img2", "img12"]
    assert natural_sort(["x2", "x2", "x1"]) == ["x1", "x2", "x2"]
    assert natural_sort(["b", "a", "c"]) == ["a", "b", "c"]
    print("PASS")

_check()
```

**Editorial:** `re.split(r'(\d+)', s)` breaks each string into alternating text and digit chunks (the capturing group keeps the digits). Casting the digit chunks to `int` makes the `key` a list that compares text lexicographically but numbers numerically, so `sorted` yields human-friendly "natural" order. Python's element-wise list comparison does the rest — no custom comparator needed. (Chunk positions alternate str/int by construction, so like compares with like.)
