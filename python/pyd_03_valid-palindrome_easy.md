## challenge: Valid Palindrome
tags: string, two-pointers
track: python
lang: python
difficulty: easy

Given a string `s`, return `True` if it reads the same forward and backward after you remove all non-alphanumeric characters and lowercase the rest.

Constraints: `0 <= len(s) <= 2 * 10^5`, `s` may contain letters, digits, spaces, and punctuation.

Example: `s = "A man, a plan, a canal: Panama"` → `True`. `s = "race a car"` → `False`.

hint: Only letters and digits count; case does not matter.
hint: Two pointers walking inward from both ends avoid building a cleaned copy.
hint: Skip any non-alphanumeric character on either side, then compare the two ends lowercased.

```python
# starter
def is_palindrome(s):
    ...
```

```python
def is_palindrome(s):
    i, j = 0, len(s) - 1
    while i < j:
        while i < j and not s[i].isalnum():
            i += 1
        while i < j and not s[j].isalnum():
            j -= 1
        if s[i].lower() != s[j].lower():
            return False
        i += 1
        j -= 1
    return True
```

```python
# harness
#__USER__
def _check():
    assert is_palindrome("A man, a plan, a canal: Panama") is True
    assert is_palindrome("race a car") is False
    assert is_palindrome("") is True
    assert is_palindrome(" ") is True
    assert is_palindrome("0P") is False
    assert is_palindrome("Was it a car or a cat I saw?") is True
    print("PASS")

_check()
```

**Editorial:** Keep two indices, one at each end. Advance them past any character that is not alphanumeric, then compare the two characters case-insensitively; a mismatch means it is not a palindrome. Move both inward and repeat until they cross. This runs in O(n) time and O(1) extra space, avoiding the O(n) memory of first filtering the string into a new list and reversing it.
