## challenge: Group Anagrams
tags: hash-table, string, sorting
track: python
lang: python
difficulty: medium

Given a list of strings `strs`, group together all strings that are anagrams of one another. Return the groups as a list of lists in any order; the strings within each group may also be in any order.

Constraints: `0 <= len(strs) <= 10^4`, strings consist of lowercase letters and may be empty.

Example: `strs = ["eat", "tea", "tan", "ate", "nat", "bat"]` → `[["bat"], ["nat", "tan"], ["ate", "eat", "tea"]]` (order not significant).

hint: Anagrams share a canonical form — something identical for every member of a group.
hint: Sorting a word's letters, or its 26-length letter-count, makes a stable dictionary key.
hint: Bucket each word under its canonical key in a dict, then return the buckets' values.

```python
# starter
def group_anagrams(strs):
    ...
```

```python
def group_anagrams(strs):
    from collections import defaultdict
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))
        groups[key].append(s)
    return list(groups.values())
```

```python
# harness
#__USER__
def _canon(groups):
    return sorted(sorted(g) for g in groups)

def _check():
    assert _canon(group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"])) == \
        _canon([["bat"], ["nat", "tan"], ["ate", "eat", "tea"]])
    assert _canon(group_anagrams([""])) == [[""]]
    assert _canon(group_anagrams(["a"])) == [["a"]]
    assert _canon(group_anagrams([])) == []
    assert _canon(group_anagrams(["abc", "bca", "cab", "xyz"])) == \
        _canon([["abc", "bca", "cab"], ["xyz"]])
    assert _canon(group_anagrams(["ddddddddddg", "dgggggggggg"])) == \
        _canon([["ddddddddddg"], ["dgggggggggg"]])
    print("PASS")

_check()
```

**Editorial:** Every anagram of a word shares the same sorted letters, so `tuple(sorted(word))` is a canonical key that all members of a group map to. Use a `defaultdict(list)` to append each word under its key, then return the dictionary's values. Sorting each word costs O(k log k), giving O(n · k log k) total for n words of length k; an O(n · k) variant keys on a 26-slot count tuple instead.
