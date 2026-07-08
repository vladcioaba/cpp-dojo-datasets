## challenge: Valid Anagram
tags: string, hash-table
track: python
lang: python
difficulty: easy

Given two strings `s` and `t`, return `True` if `t` is an anagram of `s` — that is, `t` uses exactly the same letters as `s` with the same multiplicities, just possibly reordered.

Constraints: `0 <= len(s), len(t) <= 5 * 10^4`, strings consist of lowercase letters.

Example: `s = "anagram", t = "nagaram"` → `True`. `s = "rat", t = "car"` → `False`.

hint: Anagrams must be the same length and use each character the same number of times.
hint: A character-count comparison decides it — `collections.Counter` builds one for you.
hint: Bail out early if the lengths differ, then compare the two frequency maps.

```python
# starter
def is_anagram(s, t):
    ...
```

```python
def is_anagram(s, t):
    from collections import Counter
    if len(s) != len(t):
        return False
    return Counter(s) == Counter(t)
```

```python
# harness
#__USER__
def _check():
    assert is_anagram("anagram", "nagaram") is True
    assert is_anagram("rat", "car") is False
    assert is_anagram("a", "ab") is False
    assert is_anagram("", "") is True
    assert is_anagram("ab", "ba") is True
    assert is_anagram("aacc", "ccac") is False
    print("PASS")

_check()
```

**Editorial:** Two strings are anagrams exactly when they have identical character frequencies. Different lengths can never match, so short-circuit on that first. Otherwise build a `Counter` (a dict of char → count) for each and compare them for equality. O(n) time, O(k) space where k is the alphabet size — faster than sorting both strings, which is O(n log n).
