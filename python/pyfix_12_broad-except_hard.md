## challenge: fix: Schema bugs vanish into the total
tags: code-review, debugging, exceptions
track: python
lang: python
difficulty: hard

Code review found a bug: a report whose column was misspelled upstream produced a quietly wrong total for weeks — rows with a missing `amount` key were silently dropped, when only genuinely non-numeric amounts should be skipped. Find and fix it — keep the function signature.

hint: The skipping logic is intentional — the question is *which* failures it was meant to skip.
hint: `except Exception` catches far more than bad numbers: `KeyError`, `AttributeError`, and other symptoms of code or schema bugs.
hint: Catch exactly what a malformed amount raises — `float()` raises `ValueError` for bad strings and `TypeError` for non-numeric types — and let `KeyError` propagate.

```python
# starter
def total_amount(rows):
    """Sum each row's 'amount' field, skipping rows whose amount isn't a number."""
    total = 0.0
    for row in rows:
        try:
            total += float(row["amount"])
        except Exception:
            continue
    return total
```

```python
def total_amount(rows):
    """Sum each row's 'amount' field, skipping rows whose amount isn't a number."""
    total = 0.0
    for row in rows:
        try:
            total += float(row["amount"])
        except (ValueError, TypeError):
            continue
    return total
```

```python
# harness
#__USER__
def _check():
    rows = [{"amount": "3.5"}, {"amount": "oops"}, {"amount": "1.5"}, {"amount": None}]
    assert total_amount(rows) == 5.0

    try:
        total_amount([{"amount": "2.0"}, {"amout": "9.9"}])   # misspelled column
        raised = False
    except KeyError:
        raised = True
    assert raised, "a missing 'amount' key must raise, not be silently swallowed"
    print("PASS")

_check()
```

**Editorial:** The `try` was meant to skip rows whose amount isn't a number, but `except Exception` also swallows `KeyError` — the signature of a schema bug, not of dirty data — so structurally broken rows disappear from the total without a trace. The fix is to catch exactly what a malformed amount can raise: `float()` raises `ValueError` for unparseable strings and `TypeError` for non-numeric objects, so `except (ValueError, TypeError): continue` keeps the intended skipping while letting real bugs surface loudly. In review, treat every `except Exception`/bare `except` around more than one failable operation as a place where your own bugs go to hide — name the exceptions the documented failure mode actually produces.
