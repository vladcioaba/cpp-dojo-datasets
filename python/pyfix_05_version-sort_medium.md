## challenge: fix: Version 1.10 releases before 1.9
tags: code-review, debugging, sorting
track: python
lang: python
difficulty: medium

Code review found a bug: the changelog lists `1.10.0` *before* `1.9.0`, and `10.0.0` before `2.0.0` — newest releases are buried in the middle. Find and fix it — keep the function signature.

hint: Print the sorted output for versions with multi-digit components.
hint: What type are the things being compared, and how does that type order itself?
hint: Strings compare character by character, so `"1.10" < "1.9"` — sort by a tuple of ints instead (`key=`).

```python
# starter
def sort_versions(versions):
    """Sort dotted version strings ascending: '1.9.0' comes before '1.10.0'."""
    return sorted(versions)
```

```python
def sort_versions(versions):
    """Sort dotted version strings ascending: '1.9.0' comes before '1.10.0'."""
    return sorted(versions, key=lambda v: tuple(int(part) for part in v.split(".")))
```

```python
# harness
#__USER__
def _check():
    assert sort_versions(["1.10.0", "1.9.0", "1.2.3"]) == ["1.2.3", "1.9.0", "1.10.0"]
    assert sort_versions(["10.0.0", "2.0.0"]) == ["2.0.0", "10.0.0"]
    assert sort_versions(["0.9.1", "0.9.0"]) == ["0.9.0", "0.9.1"]
    assert sort_versions([]) == []
    print("PASS")

_check()
```

**Editorial:** The versions are strings, and strings sort lexicographically: comparing `"1.10.0"` with `"1.9.0"` reaches `'1' < '9'` at the third character and stops — so 1.10 lands before 1.9. The fix is a sort key that restores numeric meaning: split on dots and compare tuples of ints, `key=lambda v: tuple(int(p) for p in v.split("."))`. Reviewers should be suspicious whenever "numbers" arrive as strings (CSV fields, version tags, IDs) and get sorted or compared without conversion — the code looks right and even works on single-digit data.
