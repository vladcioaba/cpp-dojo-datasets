## challenge: Group Anagrams
tags: defaultdict, sorting-key
track: python
lang: python
difficulty: medium

Given a list of `words`, group together the words that are anagrams of one another. Return a list of groups; within each group keep the words in their original order.

Constraints: words contain lowercase letters; the list may be empty.

Example: `["eat", "tea", "tan", "ate", "nat", "bat"]` → groups `["eat", "tea", "ate"]`, `["tan", "nat"]`, `["bat"]`.

hint: Two words are anagrams iff their sorted letters match — that sorted string is a natural group key.
hint: `''.join(sorted(word))` turns both `"tea"` and `"eat"` into `"aet"`.
hint: `collections.defaultdict(list)` lets you `append` to a group without first checking whether the key exists.

```python
# starter
def anagram_groups(words):
    ...
```

```python
from collections import defaultdict

def anagram_groups(words):
    groups = defaultdict(list)
    for word in words:
        groups[''.join(sorted(word))].append(word)
    return list(groups.values())
```

```python
# harness
#__USER__
def _norm(groups):
    return sorted(sorted(g) for g in groups)

def _check():
    assert _norm(anagram_groups(["eat", "tea", "tan", "ate", "nat", "bat"])) == \
        [["ate", "eat", "tea"], ["bat"], ["nat", "tan"]]
    assert anagram_groups([]) == []
    assert anagram_groups(["abc"]) == [["abc"]]
    assert anagram_groups(["a", "a"]) == [["a", "a"]]
    assert _norm(anagram_groups(["ab", "ba", "cd"])) == [["ab", "ba"], ["cd"]]
    print("PASS")

_check()
```

**Editorial:** The sorted-letter string is a canonical key shared by all anagrams, so grouping is just bucketing by that key. `defaultdict(list)` auto-creates an empty list on first access, so `groups[key].append(word)` works without an `if key not in groups` guard — the classic idiom for building lists-in-a-dict.
