## challenge: Roman to Integer
tags: string, hash-table
track: python
lang: python
difficulty: easy

Given a valid Roman numeral string `s`, convert it to an integer. Symbols: `I=1, V=5, X=10, L=50, C=100, D=500, M=1000`. Subtractive pairs like `IV` (4), `IX` (9), `XL` (40), `CM` (900) mean a smaller symbol placed before a larger one is subtracted.

Constraints: `1 <= len(s) <= 15`, `s` is a valid Roman numeral in `[1, 3999]`.

Example: `s = "MCMXCIV"` → `1994` (M=1000, CM=900, XC=90, IV=4).

hint: Map each symbol to its value with a dict, then scan left to right.
hint: The only twist is the subtractive rule — when does a symbol count as negative?
hint: If a symbol's value is smaller than the value of the symbol to its right, subtract it; otherwise add it.

```python
# starter
def roman_to_int(s):
    ...
```

```python
def roman_to_int(s):
    vals = {"I": 1, "V": 5, "X": 10, "L": 50, "C": 100, "D": 500, "M": 1000}
    total = 0
    for i, ch in enumerate(s):
        v = vals[ch]
        if i + 1 < len(s) and v < vals[s[i + 1]]:
            total -= v
        else:
            total += v
    return total
```

```python
# harness
#__USER__
def _check():
    assert roman_to_int("III") == 3
    assert roman_to_int("IV") == 4
    assert roman_to_int("IX") == 9
    assert roman_to_int("LVIII") == 58
    assert roman_to_int("MCMXCIV") == 1994
    assert roman_to_int("MMXXVI") == 2026
    assert roman_to_int("I") == 1
    print("PASS")

_check()
```

**Editorial:** One pass with a symbol-to-value dict. The subtractive notation collapses to a single local rule: a symbol strictly smaller than its right neighbor contributes negatively (`IV` = -1 + 5). No pair table needed. O(n) time, O(1) space.
