## challenge: String Compression
tags: string, two-pointers
track: python
lang: python
difficulty: medium

Given a string `s`, compress it by replacing each run of consecutive repeated characters with the character followed by the run length — but omit the count when the run length is 1. Return the compressed string. `"aabbccc"` becomes `"a2b2c3"`; `"abc"` stays `"abc"`. Runs longer than 9 keep their full multi-digit count.

Constraints: `0 <= len(s) <= 10^5`, `s` consists of letters (case-sensitive).

Example: `s = "abbbbbbbbbbbb"` → `"ab12"` (one `a`, twelve `b`s).

hint: Walk the string with two indices: `i` marks the start of the current run, `j` scans forward while characters match.
hint: The run length is `j - i`; append the character, then the count only if it exceeds 1.
hint: Build the pieces in a list and `"".join` at the end — repeated string concatenation is quadratic.

```python
# starter
def compress(s):
    ...
```

```python
def compress(s):
    out = []
    i = 0
    while i < len(s):
        j = i
        while j < len(s) and s[j] == s[i]:
            j += 1
        out.append(s[i])
        if j - i > 1:
            out.append(str(j - i))
        i = j
    return "".join(out)
```

```python
# harness
#__USER__
def _check():
    assert compress("aabbccc") == "a2b2c3"
    assert compress("a") == "a"
    assert compress("abbbbbbbbbbbb") == "ab12"
    assert compress("abc") == "abc"
    assert compress("") == ""
    assert compress("aaAAaa") == "a2A2a2"
    assert compress("aaaaaaaaaaaa") == "a12"
    print("PASS")

_check()
```

**Editorial:** Two-pointer run-length encoding: `i` anchors the run, `j` races ahead until the character changes, and `j - i` is the count. Emitting into a list and joining once avoids O(n²) string concatenation. Edge cases that trip people up: single-character runs (no count emitted), multi-digit counts, and the empty string. O(n) time, O(n) space.
