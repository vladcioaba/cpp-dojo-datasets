## challenge: fix: Every currency converts at the yen rate
tags: code-review, debugging, closures
track: python
lang: python
difficulty: medium

Code review found a bug: converting 10 USD to euros returns 1550 — every converter in the map applies whichever rate happened to be last in the dict. Find and fix it — keep the function signature.

hint: Each lambda looks correct alone; the problem is what they all have in common.
hint: Closures capture variables, not the values those variables had when the lambda was created.
hint: All the lambdas share the single loop variable `rate`, which ends the loop at its final value — freeze it per iteration with a default argument (`lambda usd, rate=rate: ...`).

```python
# starter
def build_converters(rates):
    """Map currency code -> function that converts a USD amount to that currency."""
    converters = {}
    for code, rate in rates.items():
        converters[code] = lambda usd: usd * rate
    return converters
```

```python
def build_converters(rates):
    """Map currency code -> function that converts a USD amount to that currency."""
    converters = {}
    for code, rate in rates.items():
        converters[code] = lambda usd, rate=rate: usd * rate
    return converters
```

```python
# harness
#__USER__
def _check():
    conv = build_converters({"eur": 0.9, "gbp": 0.8, "jpy": 155.0})
    assert abs(conv["eur"](10) - 9.0) < 1e-9, "eur converter used the wrong rate"
    assert abs(conv["gbp"](10) - 8.0) < 1e-9, "gbp converter used the wrong rate"
    assert abs(conv["jpy"](10) - 1550.0) < 1e-9
    assert abs(conv["eur"](0) - 0.0) < 1e-9
    print("PASS")

_check()
```

**Editorial:** Python closures are late-binding: each lambda closes over the loop *variable* `rate`, not the value it held when the lambda was built. The loop finishes with `rate == 155.0`, so every converter reads that final value at call time. The standard fix is to capture the current value as a default argument — `lambda usd, rate=rate: usd * rate` — since defaults are evaluated at definition time; `functools.partial(operator.mul, rate)` or a factory function also work. Reviewers should flag any lambda or `def` created inside a loop that references the loop variable.
