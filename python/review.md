## quiz: review: Everyone joins the same team
tags: code-review, functions
track: python
difficulty: easy

You're reviewing this code. What's the bug?

```python
def register_player(name, roster=[]):
    roster.append(name)
    return roster

team_red = register_player("ann")
team_blue = register_player("bob")
print(team_blue)   # ['ann', 'bob'] -- bob is on ann's team?!
```

- [ ] `append` returns `None`, so the function should `return roster + [name]` instead
- [x] the default `roster=[]` is evaluated once at `def` time, so every call that omits `roster` shares (and keeps appending to) the same list
- [ ] lists are passed by value in Python, so appends inside the function never reach the caller
- [ ] `team_blue` aliases `team_red` because assignment copies references between variables

> Default parameter values are created once, when the `def` executes — not per call. Both calls that omit `roster` get the very same list object, which accumulates every player ever registered. Use a sentinel: `def register_player(name, roster=None):` and `if roster is None: roster = []`.

## quiz: review: The session that was never found
tags: code-review, identity
track: python
difficulty: easy

You're reviewing this code. What's the bug?

```python
def find_token(sessions, user_id):
    for s in sessions:
        if s["user_id"] is user_id:
            return s["token"]
    return None

sessions = [{"user_id": 1001, "token": "t-red"},
            {"user_id": 1002, "token": "t-blue"}]
uid = int("1001")                  # parsed from a request
print(find_token(sessions, uid))   # None -- but the session exists!
```

- [ ] `int("1001")` returns a numeric string, which never equals the integer `1001`
- [ ] dict lookup with `s["user_id"]` returns `None` because the key was stored as a string
- [x] `is` tests object identity, not equality — two equal ints above 256 are usually distinct objects, so the comparison is `False`; use `==`
- [ ] the loop should use `enumerate` — without it, `s` is the key, not the dict

> `is` asks "are these the same object?", not "do they have the same value?". CPython caches only small ints (-5..256); `1001` from the literal and `1001` parsed at runtime are different objects, so `is` is `False` even though `==` is `True`. Reserve `is` for singletons like `None`; compare values with `==`.

## quiz: review: Half a star, gone
tags: code-review, arithmetic
track: python
difficulty: easy

You're reviewing this code. What's the bug?

```python
def average_rating(ratings):
    return sum(ratings) // len(ratings)

def show(product, ratings):
    return f"{product}: {average_rating(ratings)} stars"

print(show("mouse", [4, 5, 5, 4]))   # 'mouse: 4 stars' -- true mean is 4.5
```

- [ ] the f-string formats numbers with zero decimal places by default, dropping the .5
- [ ] `sum(ratings)` needs a float start value (`sum(ratings, 0.0)`) to avoid integer math
- [x] `//` is floor division — `18 // 4` is `4`, so the fractional part of the mean is silently discarded; use `/`
- [ ] `average_rating` only works for an even number of ratings

> `//` floor-divides and quietly throws away the remainder, so every average is rounded down to a whole star. The mean of `[4, 5, 5, 4]` is 4.5, but `18 // 4` yields 4. Use true division `/` (and format as needed); reach for `//` only when you specifically want a floored integer.

## quiz: review: strip() ate the filename
tags: code-review, strings
track: python
difficulty: easy

You're reviewing this code. What's the bug?

```python
def display_name(filename):
    return filename.strip(".txt")

for f in ["report.txt", "notes.txt", "test.txt"]:
    print(display_name(f))
# repor
# notes
# es
```

- [ ] `strip` only removes whitespace; its argument is ignored
- [ ] `strip` returns `None` when the suffix is absent, so some names vanish
- [ ] `strip` is case-sensitive, which is why only some names are damaged
- [x] `strip(".txt")` treats the argument as a *set of characters* to remove from both ends, not a suffix — use `removesuffix(".txt")`

> `str.strip(chars)` peels off any run of the given characters from *both* ends: for `"test.txt"` it strips leading/trailing `t`, `x`, `.` until it hits a character outside the set, leaving `"es"`. That's why some names look fine and others are mangled — a classically intermittent review trap. Use `filename.removesuffix(".txt")` (3.9+) or check `endswith` and slice.

## quiz: review: Everyone gets the staff discount
tags: code-review, closures
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
def make_discounts(rates):
    fns = []
    for rate in rates:
        fns.append(lambda price: price * (1 - rate))
    return fns

standard, member, staff = make_discounts([0.05, 0.20, 0.50])
print(standard(100.0), member(100.0), staff(100.0))
# 50.0 50.0 50.0 -- every tier gets 50% off
```

- [ ] `lambda` cannot see loop variables, so `rate` is always the first value, 0.05
- [x] each lambda closes over the *variable* `rate`, not its value at append time; when called after the loop, all three read the final value 0.50 — bind it with `lambda price, rate=rate: ...`
- [ ] `fns.append` stores the result of calling the lambda, not the lambda itself
- [ ] tuple unpacking assigns the functions in reverse order, so `standard` received the staff lambda

> Closures capture variables, not values. All three lambdas share the single loop variable `rate`, and by the time any of them runs the loop has finished with `rate == 0.50`. Freeze the current value per iteration with a default argument (`lambda price, rate=rate: ...`) or `functools.partial`.

## quiz: review: The spam that got away
tags: code-review, lists
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
def remove_banned(words, banned):
    for w in words:
        if w in banned:
            words.remove(w)
    return words

chat = ["hi", "spam", "spam", "team", "spam"]
print(remove_banned(chat, {"spam"}))
# ['hi', 'team', 'spam'] -- one slipped through
```

- [ ] `words.remove(w)` raises `ValueError` when there are duplicates, aborting the loop early
- [ ] membership tests against a set only match the first occurrence of each value
- [x] removing items from the list being iterated shifts later elements left while the iterator's index marches on, so the element right after each removal is never examined
- [ ] `remove` deletes every occurrence at once, so the loop's bookkeeping is off by the number of duplicates

> The list iterator works by index. When `remove` deletes an element, everything after it shifts left one slot, but the iterator still advances — so the element that slid into the current position is skipped. Consecutive matches therefore survive. Iterate over a copy (`for w in words[:]`) or, better, rebuild: `words[:] = [w for w in words if w not in banned]`.

## quiz: review: The checkpoint that time-travels
tags: code-review, copying
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
import copy

def checkpoint(grid):
    return copy.copy(grid)

grid = [[0, 0, 0], [0, 0, 0]]
saved = checkpoint(grid)
grid[1][2] = 7          # play a move after saving
print(saved[1][2])      # 7 -- the checkpoint changed retroactively
```

- [ ] `copy.copy` returns the same object, so `saved is grid`
- [x] `copy.copy` is shallow: it duplicates the outer list but both grids still share the same row objects, so mutating a cell shows through — use `copy.deepcopy` (or copy each row)
- [ ] item assignment `grid[1][2] = 7` rebinds the whole row, detaching it from the copy
- [ ] the fix is to return `grid[:]` — slicing is the only way to get a real copy

> A shallow copy creates a new outer list whose elements are the *same* inner row objects. `saved[1]` and `grid[1]` are one list, so writing a cell "changes the past". Deep-copy nested structures (`copy.deepcopy(grid)`) or copy each level explicitly (`[row[:] for row in grid]`); note `grid[:]` and `list(grid)` are just as shallow as `copy.copy`.

## quiz: review: The groups that never formed
tags: code-review, dicts
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
def group_by_team(pairs):
    teams = {}
    for name, team in pairs:
        teams.get(team, []).append(name)
    return teams

roster = [("ann", "red"), ("bob", "blue"), ("cai", "red")]
print(group_by_team(roster))   # {} -- everything vanished
```

- [ ] `.append` on the result of `.get` raises `AttributeError` for missing keys, which aborts silently
- [ ] the dict is keyed by team but `get` looks the value up by player name
- [x] `get(team, [])` returns a brand-new list that is never stored in the dict, so every `append` lands in an object that's immediately discarded — use `setdefault` or `defaultdict(list)`
- [ ] dicts can't hold list values without wrapping them in `tuple` first

> `dict.get` merely *returns* the default; it doesn't insert it. Each iteration builds a fresh `[]`, appends one name to it, and drops it — the dict never gains a key. `teams.setdefault(team, []).append(name)` stores the list on first use, or use `collections.defaultdict(list)`.

## quiz: review: 100 points, no podium
tags: code-review, sorting
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
def podium(results):
    scores = [line.split(",")[1] for line in results]
    return sorted(scores, reverse=True)[:3]

results = ["ann,9", "bob,100", "cai,25", "dan,7"]
print(podium(results))   # ['9', '7', '25'] -- where did 100 go?
```

- [ ] `reverse=True` sorts strings ascending because the comparison is inverted twice
- [ ] `split(",")[1]` grabs the player name, not the score
- [ ] slicing `[:3]` takes the bottom three; the top three need `[-3:]`
- [x] the scores are still strings, so they sort lexicographically (`'9' > '100'` because `'9' > '1'`) — convert to int, or pass `key=int`

> Splitting a CSV line yields strings, and strings compare character by character: `'9'` beats `'100'` because `'9' > '1'`. The sort runs fine and looks plausible on small same-width data, then misranks the moment digits differ in count. Convert early (`int(line.split(",")[1])`) or sort with `key=int`.

## quiz: review: The sum of an empty tank
tags: code-review, generators
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
def stats(readings):
    valid = (r for r in readings if r >= 0)
    n = sum(1 for _ in valid)
    total = sum(valid)
    return n, total

print(stats([3.0, -1.0, 5.0]))   # (2, 0) -- two readings totalling zero?
```

- [ ] generator expressions always skip their first element on the second pass
- [x] the counting pass exhausts the generator, so `sum(valid)` iterates over nothing and returns 0 — materialize once with `list(...)` and reuse it
- [ ] `sum(1 for _ in valid)` consumes only the truthy readings, leaving the rest for `total`
- [ ] the filter should be `r > 0`; including zero readings zeroes out the total

> A generator is a one-shot stream: once the counting loop pulls every item, it's empty forever, so the second `sum` sees nothing and returns 0. Either materialize (`valid = [r for r in readings if r >= 0]`) and reuse the list, or compute both aggregates in a single pass.

## quiz: review: Zero retries means five retries
tags: code-review, truthiness
track: python
difficulty: medium

You're reviewing this code. What's the bug?

```python
def retry_limit(config):
    return config.get("retries") or 5

print(retry_limit({"retries": 0}))   # 5 -- user explicitly disabled retries
print(retry_limit({}))               # 5
```

- [ ] `config.get` returns `False` (not `None`) for missing keys, confusing the `or`
- [x] `or` returns its left operand unless it's falsy — and `0` is falsy — so an explicit `retries: 0` is indistinguishable from a missing key; use `config.get("retries", 5)` or an `is None` check
- [ ] `or` evaluates both sides and returns whichever is larger
- [ ] the default belongs on the left: `5 or config.get("retries")`

> `x or y` doesn't mean "x, defaulting to y when absent" — it means "y whenever x is falsy", and `0`, `""`, `[]` are all falsy. A user who deliberately set `retries: 0` silently gets 5. Only `None` should trigger the default: `config.get("retries", 5)`, or fetch then test `if value is None`.

## quiz: review: The cleaner that cleans nothing
tags: code-review, exceptions
track: python
difficulty: hard

You're reviewing this code. What's the bug?

```python
def normalize_emails(entries):
    out = []
    for e in entries:
        try:
            out.append(e.strip().lowercase())
        except Exception:
            pass   # skip bad entries
    return out

print(normalize_emails(["Ann@Example.COM ", "bob@test.org"]))  # []
```

- [ ] `strip()` fails on strings that have no surrounding whitespace, so those entries are skipped
- [ ] `pass` should be `continue`; without it the loop exits after the first bad entry
- [x] `except Exception` silently swallows the `AttributeError` from the typo `lowercase` (the method is `str.lower`), so *every* entry is "skipped" and the code bug never surfaces — narrow the except to what can legitimately fail
- [ ] `out.append` returns `None`, so the appended values are lost

> There is no `str.lowercase`; every iteration raises `AttributeError`, and the blanket `except Exception: pass` converts that programming error into silently empty output. Broad excepts don't just skip bad data — they bury your own bugs. Catch the specific exceptions the data can cause (here: none for stripping/lowering a str), and let everything else propagate.

## quiz: review: A cent short at the register
tags: code-review, shadowing
track: python
difficulty: hard

You're reviewing this code. What's the bug?

```python
def round(x, ndigits=0):
    """Fast rounding without float noise."""
    p = 10 ** ndigits
    return int(x * p) / p

def total_with_tax(subtotal, rate):
    return round(subtotal * (1 + rate), 2)

print(total_with_tax(19.99, 0.0825))   # 21.63 -- register says 21.64
```

- [ ] `10 ** ndigits` is integer math and overflows for `ndigits=2`
- [ ] floats can't represent 19.99 exactly, so the bug is in the literal, not the code
- [x] the helper shadows the built-in `round` with different semantics — `int()` truncates toward zero instead of rounding to nearest — so every total is systematically up to a cent low; rename the helper (and don't reimplement rounding via `int`)
- [ ] `total_with_tax` runs before the module finishes loading, so it still calls the builtin

> Defining a module-level `round` shadows the builtin for the whole module, and this version truncates (`int(2163.917...) == 2163`) instead of rounding to nearest, so `21.639...` becomes `21.63` rather than `21.64`. Shadowing a builtin with subtly different behavior is a silent, module-wide change — reviewers should flag any def/assignment that reuses a builtin name; if custom rounding is really needed, give it its own name.
