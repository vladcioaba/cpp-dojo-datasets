## challenge: fix: Failing scores dodge the purge
tags: code-review, debugging, lists
track: python
lang: python
difficulty: medium

Code review found a bug: purging scores below the passing mark leaves some failing scores behind — always ones that sat right next to another failing score. Find and fix it — keep the function signature (and keep it in-place: callers hold references to the list).

hint: Trace the loop by hand on `[50, 40, 90]` with passing=60 and watch the indices.
hint: The list is being modified while it is being iterated.
hint: `pop(i)` shifts everything after `i` one slot left, but the iterator still advances — the element right after each removal is never examined. Rebuild and assign back via `scores[:] = ...`.

```python
# starter
def drop_failing(scores, passing):
    """Remove every score below `passing` from the list, in place. Returns the list."""
    for i, s in enumerate(scores):
        if s < passing:
            scores.pop(i)
    return scores
```

```python
def drop_failing(scores, passing):
    """Remove every score below `passing` from the list, in place. Returns the list."""
    scores[:] = [s for s in scores if s >= passing]
    return scores
```

```python
# harness
#__USER__
def _check():
    data = [50, 40, 90, 30, 20, 95]
    out = drop_failing(data, 60)
    assert out == [90, 95], out
    assert out is data, "must modify the caller's list in place"
    assert drop_failing([10, 10, 10], 60) == []
    assert drop_failing([70, 80], 60) == [70, 80]
    print("PASS")

_check()
```

**Editorial:** Removing elements from a list while iterating over it desynchronizes the iterator from the data: `pop(i)` shifts every later element left, then the loop advances anyway, so the element that slid into position `i` is never inspected. That's why adjacent failing scores survive in pairs. The fix filters into a fresh list and assigns it back through a slice — `scores[:] = [s for s in scores if s >= passing]` — which preserves the in-place contract (same list object) while never mutating mid-iteration. A reviewer's rule of thumb: any `remove`/`pop`/`del` on the sequence named in the `for` header is a bug until proven otherwise.
