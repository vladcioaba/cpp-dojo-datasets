## challenge: Longest Substring Without Repeating Characters
tags: string, sliding-window, hash-table
track: python
lang: python
difficulty: hard

Given a string `s`, return the length of the longest contiguous substring that contains no repeated character.

Constraints: `0 <= len(s) <= 5 * 10^4`, `s` may contain letters, digits, symbols, and spaces.

Example: `s = "abcabcbb"` → `3` (the substring `"abc"`). `s = "pwwkew"` → `3` (the substring `"wke"`).

hint: Maintain a window that always holds distinct characters and slide it right.
hint: Remember the last index where each character appeared.
hint: On a repeat inside the window, jump the window's left edge just past the previous occurrence.

```python
# starter
def length_of_longest_substring(s):
    ...
```

```python
def length_of_longest_substring(s):
    last = {}
    start = 0
    best = 0
    for i, c in enumerate(s):
        if c in last and last[c] >= start:
            start = last[c] + 1
        last[c] = i
        best = max(best, i - start + 1)
    return best
```

```python
# harness
#__USER__
def _check():
    assert length_of_longest_substring("abcabcbb") == 3
    assert length_of_longest_substring("bbbbb") == 1
    assert length_of_longest_substring("pwwkew") == 3
    assert length_of_longest_substring("") == 0
    assert length_of_longest_substring(" ") == 1
    assert length_of_longest_substring("dvdf") == 3
    assert length_of_longest_substring("abba") == 2
    print("PASS")

_check()
```

**Editorial:** Slide a window `[start, i]` across the string while recording each character's most recent index in a dict. When the current character was seen at a position at or after `start`, that repeat is inside the window, so move `start` to just past the previous occurrence. The window is always duplicate-free, and its largest width is the answer. Each character is visited once: O(n) time, O(min(n, alphabet)) space. The `last[c] >= start` guard is what correctly handles cases like `"abba"`.
