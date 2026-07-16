## challenge: fix: Leaderboard ties sorted backwards
tags: code-review, debugging, sorting
track: python
lang: python
difficulty: hard

Code review found a bug: players tied on score appear in reverse alphabetical order (Z→A) on the leaderboard, though the spec says ties break A→Z. Find and fix it — keep the function signature.

hint: The scores are ordered correctly; only the tie-break direction is wrong.
hint: What does `reverse=True` apply to — one component of the key, or the whole comparison?
hint: `reverse=True` flips the entire key, tie-breakers included. Encode direction in the key itself: negate the numeric part (`(-score, name)`) and drop `reverse`.

```python
# starter
def leaderboard(players):
    """Sort by score, highest first; ties broken by name A->Z."""
    return sorted(players, key=lambda p: (p["score"], p["name"]), reverse=True)
```

```python
def leaderboard(players):
    """Sort by score, highest first; ties broken by name A->Z."""
    return sorted(players, key=lambda p: (-p["score"], p["name"]))
```

```python
# harness
#__USER__
def _check():
    players = [
        {"name": "mia", "score": 90},
        {"name": "alex", "score": 90},
        {"name": "zoe", "score": 95},
    ]
    names = [p["name"] for p in leaderboard(players)]
    assert names == ["zoe", "alex", "mia"], names

    players = [
        {"name": "dan", "score": 70},
        {"name": "bea", "score": 70},
        {"name": "cy", "score": 70},
    ]
    names = [p["name"] for p in leaderboard(players)]
    assert names == ["bea", "cy", "dan"], names

    assert leaderboard([]) == []
    print("PASS")

_check()
```

**Editorial:** `reverse=True` reverses the *entire* sort order — every component of the key tuple — so while scores correctly come out highest-first, tied names come out Z→A as well. Mixed-direction sorts must encode direction inside the key: negate the numeric component (`key=lambda p: (-p["score"], p["name"])`) and sort forward. (For non-negatable keys, the alternative is two stable passes: sort by the secondary key first, then by the primary.) This is a cmp-era habit — thinking of `reverse` as "flip my main criterion" — and reviewers should re-check every `reverse=True` that rides along with a multi-part key.
