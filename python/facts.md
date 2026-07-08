## fact: Mutable default arguments are evaluated once
tags: functions, gotcha
track: python

A default argument is evaluated a single time — when the `def` executes, not on each call. If the default is mutable (`[]`, `{}`, `set()`), every call that omits the argument shares the *same* object, so mutations leak across calls.

The fix is a sentinel: default to `None` and build a fresh object inside the body.

```python
def bad(x, acc=[]):
    acc.append(x)
    return acc
bad(1)            # [1]
bad(2)            # [1, 2]  <-- same list!

def good(x, acc=None):
    acc = [] if acc is None else acc
    acc.append(x)
    return acc
```

## fact: list vs tuple — mutability and hashability
tags: data-structures, core
track: python

`list` is mutable and variable-length; `tuple` is immutable and typically fixed-shape ("a record"). Because a tuple's contents can't be reassigned, a tuple of hashable items is itself hashable, so it can be a `dict` key or `set` member — a list never can. This mirrors string immutability: strings, like tuples, are hashable and safe to share.

Note "immutable" means the *tuple* can't be rebound, not that a mutable element inside it is frozen.

```python
{(1, 2): "point"}          # fine — tuple key
# {[1, 2]: "point"}        # TypeError: unhashable type: 'list'
t = (1, [2, 3])
t[1].append(4)             # allowed — the inner list is mutable
```

## fact: is checks identity, == checks equality
tags: operators, gotcha
track: python

`==` asks "do these have the same value?" (calls `__eq__`). `is` asks "are these the exact same object in memory?" (compares `id()`). They coincide often enough to hide bugs. Use `is` only for singletons — `None`, `True`, `False` — never for value comparison.

```python
a = [1, 2]
b = [1, 2]
a == b     # True  — equal values
a is b     # False — distinct objects
x = None
x is None  # correct idiom (not x == None)
```

## fact: Small integers and short strings are cached
tags: internals, gotcha
track: python

CPython pre-creates and reuses the integer objects from -5 to 256, so `is` on values in that range is accidentally `True`. Outside it, equal ints are usually distinct objects. The compiler also interns string literals that look like identifiers and de-duplicates constants within one code object. None of this is guaranteed by the language — so never rely on `is` for numbers or strings.

```python
a = 256; b = int("256")
a is b            # True  — cached
c = 257; d = int("257")
c is d            # False — not cached
```

## fact: The GIL serializes Python bytecode
tags: concurrency, internals
track: python

CPython's Global Interpreter Lock lets only one thread execute Python bytecode at a time. Threads still help with I/O-bound work (the GIL is released during blocking calls), but they give no speedup for CPU-bound pure-Python work. For CPU parallelism use `multiprocessing`, a C extension that releases the GIL, or a free-threaded build. The GIL also does *not* make your code automatically thread-safe: `+=` on a shared counter is still a race.

```python
# CPU-bound: threads won't scale, processes will
from multiprocessing import Pool
with Pool() as p:
    results = p.map(heavy_compute, work)   # true parallelism
```

## fact: Generators and iterators are lazy and one-shot
tags: iterators, laziness
track: python

An *iterable* is anything you can loop over (has `__iter__`); an *iterator* is the cursor that yields items one at a time (has `__next__`, raises `StopIteration` when done). A generator function (uses `yield`) or a generator expression produces an iterator: values are computed on demand and never stored, so memory stays flat even over huge sequences. But an iterator is exhausted after one pass — iterate again and you get nothing.

```python
def squares(n):
    for i in range(n):
        yield i * i        # lazy: one value at a time
g = squares(3)
list(g)     # [0, 1, 4]
list(g)     # []  — already exhausted
```

## fact: Comprehensions build collections in one expression
tags: comprehensions, idioms
track: python

List, set, and dict comprehensions (and generator expressions) replace explicit append loops with a single readable expression, and run faster because the loop stays in C. Swap `[]` for `()` to get a lazy generator instead of a materialized list. Pair with `enumerate` (index + value) and `zip` (parallel iteration) for common patterns.

```python
[x * x for x in range(5) if x % 2]      # [1, 9]      list
{c for c in "banana"}                    # {'a','b','n'} set
{k: v for k, v in zip("ab", [1, 2])}     # {'a':1,'b':2} dict
(x * x for x in range(5))                # lazy generator
for i, c in enumerate("ab"): ...         # 0 'a', 1 'b'
```

## fact: *args and **kwargs capture variadic arguments
tags: functions, core
track: python

In a signature, `*args` collects extra positional arguments into a tuple and `**kwargs` collects extra keyword arguments into a dict. At a call site the same `*`/`**` operators *unpack* an iterable/mapping into arguments. A bare `*` in a signature forces the parameters after it to be keyword-only.

```python
def f(*args, **kwargs):
    return args, kwargs
f(1, 2, x=3)              # ((1, 2), {'x': 3})

nums = [1, 2, 3]
print(*nums)              # unpack -> print(1, 2, 3)

def g(a, *, verbose=False):   # verbose is keyword-only
    ...
```

## fact: Decorators wrap a callable in another callable
tags: decorators, functions
track: python

`@dec` above a function means `func = dec(func)` — the decorator receives the function and returns a replacement, usually a wrapper that adds behavior around the original call. Use `functools.wraps` to preserve the wrapped function's name and docstring. Decorators are how `@property`, `@staticmethod`, and `functools.lru_cache` are implemented.

```python
import functools
def timed(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        # ... start timer ...
        return fn(*args, **kwargs)
    return wrapper

@timed
def work(): ...
```

## fact: Closures capture variables, not values (late binding)
tags: closures, gotcha
track: python

A closure remembers the *variable*, not the value it had when the closure was created. If you build closures in a loop over a loop variable, they all see the variable's final value — a classic surprise. Bind the current value explicitly with a default argument to freeze it per iteration.

```python
fns = [lambda: i for i in range(3)]
[f() for f in fns]                 # [2, 2, 2]  — all see final i

fns = [lambda i=i: i for i in range(3)]
[f() for f in fns]                 # [0, 1, 2]  — value frozen
```

## fact: __slots__ trades flexibility for memory
tags: internals, performance
track: python

By default each instance carries a `__dict__` so you can attach arbitrary attributes. Declaring `__slots__` replaces that dict with fixed descriptor storage: instances use noticeably less memory and attribute access is a touch faster, but you can no longer set attributes that aren't listed (and there's no `__dict__` unless you add one).

```python
class Point:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x, self.y = x, y
p = Point(1, 2)
p.z = 3        # AttributeError — 'z' not in __slots__
```

## fact: Dunder methods hook into language syntax
tags: data-model, protocols
track: python

Double-underscore ("dunder") methods let your objects respond to built-in syntax and functions: `__init__` (construction), `__repr__`/`__str__` (display), `__eq__`/`__hash__` (equality/hashing), `__len__`, `__getitem__`, `__iter__`, `__call__`, `__enter__`/`__exit__`. Python's operators and builtins are defined in terms of these protocols — implement the dunder and the syntax just works.

```python
class Vec:
    def __init__(self, x, y): self.x, self.y = x, y
    def __add__(self, o):     return Vec(self.x + o.x, self.y + o.y)
    def __repr__(self):       return f"Vec({self.x}, {self.y})"
Vec(1, 2) + Vec(3, 4)          # Vec(4, 6)
```

## fact: Duck typing cares about behavior, not type
tags: typing, philosophy
track: python

"If it walks like a duck and quacks like a duck, treat it as a duck." Python code generally doesn't check an object's class — it just calls the methods it needs and works with anything that supplies them. This is why a function that iterates works on lists, tuples, generators, files, and any custom object with `__iter__`. Prefer relying on a protocol over `isinstance` checks.

```python
def total(items):        # any iterable of numbers works
    return sum(items)
total([1, 2, 3])         # list
total((1, 2, 3))         # tuple
total(x for x in range(4))   # generator
```

## fact: EAFP over LBYL
tags: idioms, exceptions
track: python

Pythonic style prefers EAFP — "Easier to Ask Forgiveness than Permission": attempt the operation and handle the exception if it fails. The alternative, LBYL ("Look Before You Leap"), guards with checks first, which both duplicates work and opens a race window (the state can change between the check and the use).

```python
# LBYL — has a check-then-act gap
if key in d:
    v = d[key]

# EAFP — one atomic attempt
try:
    v = d[key]
except KeyError:
    v = default
```

## fact: Shallow copy duplicates one level; deep copy recurses
tags: copying, gotcha
track: python

`list(x)`, `x[:]`, and `copy.copy` make a *shallow* copy: a new outer container whose elements are the *same* inner objects. Mutating a nested object shows up in both copies. `copy.deepcopy` recursively clones everything, so the copy is fully independent (at the cost of speed and shared-structure duplication).

```python
import copy
a = [[1, 2], [3, 4]]
b = copy.copy(a)         # shallow
b[0].append(99)
a                        # [[1, 2, 99], [3, 4]]  — leaked!

c = copy.deepcopy(a)     # fully independent
```

## fact: collections has better containers than dict and list
tags: collections, stdlib
track: python

`defaultdict(factory)` auto-creates missing values (no `KeyError`, no `setdefault` boilerplate). `Counter` tallies hashables and gives `.most_common()`. `deque` is a double-ended queue with O(1) appends/pops at *both* ends — use it as a queue or a bounded sliding window instead of `list.pop(0)`, which is O(n).

```python
from collections import defaultdict, Counter, deque
g = defaultdict(list)
g["a"].append(1)                  # no KeyError
Counter("mississippi").most_common(1)   # [('i', 4)]  (i and s tie)
dq = deque([1, 2], maxlen=3)
dq.appendleft(0)                  # O(1) at the front
```

## fact: itertools composes lazy iterator building blocks
tags: itertools, laziness
track: python

`itertools` provides memory-efficient iterators you compose like a pipeline: `count`, `cycle`, `repeat` (infinite), `chain` (concatenate), `islice` (slice a lazy stream), `groupby`, `accumulate` (running totals), and the combinatorics `product`/`permutations`/`combinations`. Nothing is materialized until you consume it, so you can slice into infinite sequences.

```python
import itertools
list(itertools.islice(itertools.count(10, 5), 3))   # [10, 15, 20]
list(itertools.chain([1, 2], [3, 4]))               # [1, 2, 3, 4]
list(itertools.accumulate([1, 2, 3, 4]))            # [1, 3, 6, 10]
```

## fact: Context managers guarantee cleanup with `with`
tags: context-managers, resources
track: python

A `with` block calls `__enter__` on entry and `__exit__` on exit — including on exceptions and early returns — so resources (files, locks, connections) are released deterministically. It is Python's answer to try/finally boilerplate. Write one via `__enter__`/`__exit__` on a class, or more simply with `@contextlib.contextmanager` around a generator that `yield`s once.

```python
from contextlib import contextmanager
@contextmanager
def opened(path):
    f = open(path)
    try:
        yield f            # body runs here
    finally:
        f.close()          # always runs
with opened("data.txt") as f:
    ...
```

## fact: Slicing supports negative indices and a step
tags: sequences, slicing
track: python

`seq[start:stop:step]` returns a new sequence; `start` is inclusive, `stop` exclusive, and any part may be omitted. Negative indices count from the end (`-1` is last). A negative step walks backwards, so `seq[::-1]` reverses. Out-of-range slice bounds are clamped silently (unlike indexing, which raises). Slicing a `str` or `tuple` yields a new immutable copy — the original is never mutated.

```python
s = "abcdef"
s[1:4]      # 'bcd'
s[-2:]      # 'ef'
s[::-1]     # 'fedcba'
s[::2]      # 'ace'
```

## fact: Truthiness — many values are "falsy"
tags: booleans, gotcha
track: python

`if x:` calls `bool(x)`. Falsy values include `None`, `False`, numeric zero (`0`, `0.0`, `0j`), and every *empty* container (`""`, `[]`, `{}`, `()`, `set()`). Everything else is truthy — including the string `"0"`, `"False"`, and the list `[0]`. `and`/`or` are short-circuiting and return one of their *operands*, not a bool, which is handy for defaults.

```python
bool("0")            # True  — non-empty string
bool([0])            # True  — non-empty list
0 or "fallback"      # 'fallback'  (returns operand)
"" and "x"           # ''          (short-circuits)
```

## fact: sorted takes a key and is stable
tags: sorting, algorithms
track: python

`sorted(iterable, key=..., reverse=...)` returns a new list; `list.sort()` sorts in place. `key` maps each element to the value to compare (called once per element — the decorate-sort-undecorate pattern). Python's sort (Timsort) is *stable*: elements comparing equal keep their original relative order, so you can sort by successive keys to get multi-level ordering.

```python
sorted(["bb", "a", "ccc"], key=len)          # ['a', 'bb', 'ccc']
data = [("b", 2), ("a", 2), ("c", 1)]
sorted(data, key=lambda p: p[1])
# [('c', 1), ('b', 2), ('a', 2)]  — 'b' before 'a' preserved
```

## fact: Sets give fast membership and algebra
tags: sets, data-structures
track: python

A `set` is an unordered collection of unique hashables with O(1) average membership tests, versus O(n) for a list. Sets support mathematical operations: union `|`, intersection `&`, difference `-`, symmetric difference `^`. `frozenset` is the immutable, hashable variant (usable as a dict key or set member). Note `{}` is an empty *dict* — use `set()` for an empty set.

```python
a = {1, 2, 3}; b = {2, 3, 4}
a & b        # {2, 3}     intersection
a | b        # {1,2,3,4}  union
a - b        # {1}        difference
a ^ b        # {1, 4}     symmetric difference
```

## fact: try/except/else/finally each have a role
tags: exceptions, control-flow
track: python

`try` guards code; `except` handles specific exceptions (catch narrow types, not bare `except:`); `else` runs only if *no* exception fired (put the "success" code there so it isn't accidentally guarded); `finally` always runs for cleanup, even on `return` or re-raise. Exceptions are for exceptional flow, not routine branching, but they are cheap when not raised.

```python
try:
    v = risky()
except ValueError as e:
    handle(e)
else:
    use(v)             # only if no exception
finally:
    cleanup()          # always
```

## fact: dataclasses generate boilerplate from annotations
tags: dataclasses, stdlib
track: python

`@dataclass` reads the class's annotated fields and auto-generates `__init__`, `__repr__`, and `__eq__` (and ordering/`__hash__` with flags). It removes the tedious `self.x = x` and value-equality boilerplate. Mutable defaults must use `field(default_factory=list)` — a plain `= []` raises `ValueError` at class creation, guarding the mutable-default trap for you.

```python
from dataclasses import dataclass, field
@dataclass
class Cart:
    owner: str
    items: list = field(default_factory=list)   # not = []
Cart("me") == Cart("me")     # True — value equality
```

## fact: Type hints are annotations, not runtime enforcement
tags: typing, gotcha
track: python

Type hints (`x: int`, `-> str`) are stored in `__annotations__` and used by static checkers (mypy, pyright), IDEs, and tools like dataclasses/pydantic. CPython itself does **not** check them at runtime — passing the "wrong" type runs normally until something actually breaks. They document intent; they don't make Python statically typed.

```python
def add(a: int, b: int) -> int:
    return a + b
add("x", "y")          # 'xy'  — hints ignored at runtime
add.__annotations__    # {'a': int, 'b': int, 'return': int}
```

## fact: functools.lru_cache memoizes by arguments
tags: functools, performance
track: python

`@lru_cache(maxsize=N)` (or `@cache` for unbounded) stores results keyed by the call arguments, turning repeated or overlapping calls into O(1) lookups — it collapses exponential naive recursion (like Fibonacci) to linear. Requirements: arguments must be hashable, and the function should be pure (same args → same result), since stale side effects won't be recomputed. Inspect hits/misses via `.cache_info()`.

```python
from functools import lru_cache
@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
fib(100)               # instant — memoized
fib.cache_info()       # CacheInfo(hits=98, ...)
```
