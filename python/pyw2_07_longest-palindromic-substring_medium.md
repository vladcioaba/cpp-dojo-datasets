## challenge: Longest Palindromic Substring
tags: string, dp
track: python
lang: python
difficulty: medium

Given a string `s`, return the longest contiguous substring of `s` that is a palindrome. If several palindromic substrings tie for the longest, returning any one of them is accepted.

Constraints: `1 <= len(s) <= 1000`, `s` consists of letters and digits.

Example: `s = "babad"` → `"bab"` (`"aba"` is equally valid).

hint: Every palindrome has a center — either a single character (odd length) or a gap between two characters (even length).
hint: For each of the `2n - 1` centers, expand outward while the two ends match.
hint: Track the best window seen; expansion from all centers gives O(n²) time with O(1) extra space — no DP table required.

```python
# starter
def longest_palindrome(s):
    ...
```

```python
def longest_palindrome(s):
    best = s[0]
    for i in range(len(s)):
        for lo, hi in ((i, i), (i, i + 1)):
            while lo >= 0 and hi < len(s) and s[lo] == s[hi]:
                lo -= 1
                hi += 1
            if hi - lo - 1 > len(best):
                best = s[lo + 1:hi]
    return best
```

```python
# harness
#__USER__
def _check():
    assert longest_palindrome("babad") in ("bab", "aba")
    assert longest_palindrome("cbbd") == "bb"
    assert longest_palindrome("a") == "a"
    assert longest_palindrome("ac") in ("a", "c")
    assert longest_palindrome("bananas") == "anana"
    assert longest_palindrome("forgeeksskeegfor") == "geeksskeeg"
    assert longest_palindrome("aacabdkacaa") == "aca"
    print("PASS")

_check()
```

**Editorial:** Expand around every center: `n` odd centers `(i, i)` and `n - 1` even centers `(i, i + 1)`. Each expansion grows while the ends match; after the loop overshoots, the palindrome is `s[lo+1:hi]` with length `hi - lo - 1`. O(n²) time, O(1) space — beats the O(n²)-space DP table, and Manacher's algorithm gets O(n) if you ever need it.
