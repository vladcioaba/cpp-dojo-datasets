## challenge: Word Break
tags: dynamic-programming, trie, memoization, string
track: python
lang: python
difficulty: hard

Given a string `s` and a list of words `word_dict`, return `True` if `s` can be segmented into a space-separated sequence of one or more dictionary words. Each dictionary word may be reused any number of times.

Constraints: `1 <= len(s) <= 300`, `1 <= len(word_dict) <= 1000`, `1 <= len(word) <= 20`, all strings are lowercase letters.

Example: `s = "applepenapple", word_dict = ["apple", "pen"]` → `True` (`apple + pen + apple`).

hint: Let `dp[i]` mean "the prefix `s[:i]` can be segmented".
hint: `dp[i]` is true if some `j < i` has `dp[j]` true and `s[j:i]` is a dictionary word.
hint: Put the dictionary in a set for O(1) membership checks, and seed `dp[0] = True`.

```python
# starter
def word_break(s, word_dict):
    ...
```

```python
def word_break(s, word_dict):
    words = set(word_dict)
    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True
    for i in range(1, n + 1):
        for j in range(i):
            if dp[j] and s[j:i] in words:
                dp[i] = True
                break
    return dp[n]
```

```python
# harness
#__USER__
def _check():
    assert word_break("leetcode", ["leet", "code"]) is True
    assert word_break("applepenapple", ["apple", "pen"]) is True
    assert word_break("catsandog", ["cats", "dog", "sand", "and", "cat"]) is False
    assert word_break("a", ["a"]) is True
    assert word_break("aaaaaaa", ["aaaa", "aaa"]) is True
    assert word_break("cars", ["car", "ca", "rs"]) is True
    print("PASS")

_check()
```

**Editorial:** Prefix DP. `dp[i]` records whether `s[:i]` splits into dictionary words. Extend by checking every cut point `j`: if `s[:j]` is already segmentable and the suffix `s[j:i]` is a word, then `s[:i]` is too. A set gives O(1) lookups. O(n^2) cut points times the substring length gives roughly O(n^2 · L) time, O(n) space.
