## challenge: fix: Ten dimes don't make a dollar
tags: code-review, debugging, floats
track: python
lang: python
difficulty: hard

Code review found a bug: a customer who pays 1.00 in ten 0.10 installments is flagged as *not* having covered the bill, while other payment plans work fine. Find and fix it — keep the function signature.

hint: Print `paid` with `repr()` after summing ten 0.10 payments.
hint: 0.1 has no exact binary representation; adding it repeatedly accumulates representation error.
hint: Never compare accumulated floats with `==` — compare within a tolerance (`math.isclose`) or work in integer cents.

```python
# starter
def payments_cover(installments, total):
    """True if the installments add up to exactly the total owed."""
    paid = 0.0
    for amount in installments:
        paid += amount
    return paid == total
```

```python
import math

def payments_cover(installments, total):
    """True if the installments add up to exactly the total owed."""
    paid = 0.0
    for amount in installments:
        paid += amount
    return math.isclose(paid, total, rel_tol=1e-9, abs_tol=1e-9)
```

```python
# harness
#__USER__
def _check():
    assert payments_cover([0.1] * 10, 1.0), "ten 0.10 payments must cover 1.00"
    assert payments_cover([0.5, 0.25, 0.25], 1.0)
    assert not payments_cover([0.5, 0.25], 1.0), "underpayment must not pass"
    assert payments_cover([], 0.0)
    print("PASS")

_check()
```

**Editorial:** `0.1` cannot be represented exactly in binary floating point; each addition carries a tiny error, and after ten of them `paid` is `0.9999999999999999`, which `== 1.0` rejects. Payments of 0.5/0.25 happen to be exact powers of two, which is why "other plans work fine" — the classic intermittent float bug. Fix the comparison, not the arithmetic: `math.isclose(paid, total, rel_tol=1e-9, abs_tol=1e-9)` (the `abs_tol` matters so that totals of 0.0 still compare). For real money, the deeper fix is to avoid floats entirely — integer cents or `decimal.Decimal`. Reviewers flag any `==` between floats that were produced by accumulation.
