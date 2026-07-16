## challenge: fix: Account drained below zero
tags: code-review, debugging, race-condition
track: python
lang: python
difficulty: hard

Code review found a bug: when the notification hook triggers another transaction on the same accounts, the source balance can go negative even though the funds check passed. Find and fix it — keep the function signature.

hint: The balance check is correct — the question is what can happen *between* the check and the debit.
hint: Time-of-check to time-of-use: the hook runs arbitrary code while the check's conclusion is still pending.
hint: Make check-and-update atomic: perform the debit/credit immediately after the check, and only then run the hook (in real concurrent code, hold a lock around check+update or re-validate).

```python
# starter
def transfer(accounts, src, dst, amount, on_transfer=None):
    """Move `amount` from accounts[src] to accounts[dst] if funds allow.

    `on_transfer` is a notification hook; handlers may run arbitrary code,
    including other transactions against the same `accounts` mapping.
    """
    if accounts[src] >= amount:
        if on_transfer is not None:
            on_transfer(accounts)
        accounts[src] -= amount
        accounts[dst] += amount
        return True
    return False
```

```python
def transfer(accounts, src, dst, amount, on_transfer=None):
    """Move `amount` from accounts[src] to accounts[dst] if funds allow.

    `on_transfer` is a notification hook; handlers may run arbitrary code,
    including other transactions against the same `accounts` mapping.
    """
    if accounts[src] >= amount:
        accounts[src] -= amount
        accounts[dst] += amount
        if on_transfer is not None:
            on_transfer(accounts)
        return True
    return False
```

```python
# harness
#__USER__
def _check():
    accounts = {"a": 100, "b": 0}
    assert transfer(accounts, "a", "b", 60) is True
    assert accounts == {"a": 40, "b": 60}
    assert transfer(accounts, "a", "b", 100) is False
    assert accounts == {"a": 40, "b": 60}

    # A handler that fires a rival transaction while ours is in flight.
    accounts = {"a": 100, "b": 0, "c": 0}
    def rival(accts):
        if accts["a"] >= 100:
            accts["a"] -= 100
            accts["c"] += 100
    transfer(accounts, "a", "b", 60, on_transfer=rival)
    assert accounts["a"] >= 0, "source account overdrawn: %r" % accounts
    assert accounts["a"] + accounts["b"] + accounts["c"] == 100
    print("PASS")

_check()
```

**Editorial:** This is a time-of-check/time-of-use (TOCTOU) bug: the code verifies `accounts[src] >= amount`, then runs the hook — arbitrary foreign code that can move money — and only afterwards applies the debit. By then the check's conclusion may be stale: the rival handler drains the account between check and use, and the debit pushes it to -60. The fix makes check-and-update one uninterruptible unit — debit and credit immediately after the guard, with the hook moved outside the critical section. The same shape appears with threads (`if key not in cache: cache[key] = ...`), files (`os.path.exists` then `open`), and `await` points; the reviewer's question is always "what can run between this check and this action, and is the check still true then?"
