## quiz: What does this print?
tags: functions, gotcha
track: python

```python
def f(x, acc=[]):
    acc.append(x)
    return acc
print(f(1), f(2))
```

- [ ] `[1] [1, 2]`
- [x] `[1, 2] [1, 2]`
- [ ] `[1] [2]`
- [ ] `[1, 2] [1]`

> The default list is created once and shared across calls, so `f(1)` and `f(2)` mutate the same object. Both arguments to `print` are evaluated *before* printing, and both are references to that one list — which by then holds `[1, 2]`. Hence the same value twice.

## quiz: What does this print?
tags: internals, gotcha
track: python

```python
a = 256
b = int("256")
c = 257
d = int("257")
print(a is b)
print(c is d)
```

- [ ] `True` then `True`
- [x] `True` then `False`
- [ ] `False` then `False`
- [ ] `False` then `True`

> CPython caches the integers -5 through 256, so `256` and `int("256")` are the *same* cached object and `is` is `True`. `257` is outside the cache, so `int("257")` builds a fresh object each time and `is` is `False`. Value equality (`==`) would be `True` for both — never use `is` to compare numbers.

## quiz: What does this print?
tags: internals, strings
track: python

```python
a = "hi"
b = "".join(["h", "i"])
print(a == b, a is b)
```

- [ ] `True True`
- [x] `True False`
- [ ] `False False`
- [ ] `False True`

> The two strings are equal in value (`==` is `True`), but `b` is built at runtime by `join`, producing a distinct object, so `a is b` is `False`. Only the literal `a` is interned. Equal strings are *not* guaranteed to be the same object — `is` on strings is unreliable.

## quiz: What does this print?
tags: closures, gotcha
track: python

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])
```

- [ ] `[0, 1, 2]`
- [x] `[2, 2, 2]`
- [ ] `[0, 0, 0]`
- [ ] raises `NameError`

> Each lambda closes over the *variable* `i`, not its value at creation time. By the time the lambdas are called, the loop has finished and `i` is `2`, so all three return `2`. To capture per-iteration values, bind a default: `lambda i=i: i`.

## quiz: What does this print?
tags: iterators, laziness
track: python

```python
g = (x * x for x in range(3))
print(list(g))
print(list(g))
```

- [ ] `[0, 1, 4]` then `[0, 1, 4]`
- [x] `[0, 1, 4]` then `[]`
- [ ] `[0, 1, 4]` then raises `StopIteration`
- [ ] `[]` then `[]`

> A generator is a one-shot iterator. The first `list(g)` consumes it fully; the second finds it already exhausted and yields `[]` (it does not restart or raise). To iterate twice, rebuild the generator or materialize it into a list first.

## quiz: What does this print?
tags: lists, aliasing
track: python

```python
m = [[]] * 3
m[0].append(1)
print(m)
```

- [ ] `[[1], [], []]`
- [x] `[[1], [1], [1]]`
- [ ] `[[1]]`
- [ ] raises `IndexError`

> `[[]] * 3` replicates the *same* inner list reference three times, not three independent lists. Appending through `m[0]` mutates that shared list, so the change shows in all three slots. Use `[[] for _ in range(3)]` to get distinct lists.

## quiz: What does this print?
tags: copying, gotcha
track: python

```python
import copy
a = [[1, 2], [3, 4]]
b = copy.copy(a)
b[0].append(99)
print(a)
```

- [ ] `[[1, 2], [3, 4]]`
- [x] `[[1, 2, 99], [3, 4]]`
- [ ] `[[1, 2, 99], [3, 4, 99]]`
- [ ] raises `TypeError`

> `copy.copy` is a shallow copy: `b` is a new outer list but its elements are the *same* inner lists as `a`. Mutating `b[0]` mutates the object `a[0]` also points to, so the append is visible through `a`. `copy.deepcopy` would keep them independent.

## quiz: What does this print?
tags: dict, ordering
track: python

```python
d = {}
d['b'] = 1
d['a'] = 2
d['c'] = 3
print(list(d))
```

- [ ] `['a', 'b', 'c']`
- [x] `['b', 'a', 'c']`
- [ ] order is arbitrary and unpredictable
- [ ] `['c', 'a', 'b']`

> Since Python 3.7, `dict` preserves *insertion* order as a language guarantee — it does not sort keys. Iterating (or `list(d)`) yields keys in the order they were first added: `b`, `a`, `c`.

## quiz: What does this print?
tags: lists, mutation
track: python

```python
a = [1, 2, 3]
b = a
a += [4]
print(b)
```

- [ ] `[1, 2, 3]`
- [x] `[1, 2, 3, 4]`
- [ ] raises `TypeError`
- [ ] `[4, 1, 2, 3]`

> For lists, `a += [4]` calls `__iadd__`, which extends the list *in place* — it does not rebind `a`. Since `b` aliases the same list object, `b` sees the appended `4`. (Contrast `a = a + [4]`, which builds a new list and rebinds only `a`, leaving `b` unchanged.)

## quiz: What does this print?
tags: booleans, truthiness
track: python

```python
vals = [0, 0.0, "", "0", [], [0], None]
print([bool(v) for v in vals])
```

- [ ] `[False, False, False, False, False, False, False]`
- [x] `[False, False, False, True, False, True, False]`
- [ ] `[False, False, False, True, False, False, False]`
- [ ] `[False, False, True, True, False, True, False]`

> Falsy: `0`, `0.0`, empty string `""`, empty list `[]`, and `None`. Truthy: the string `"0"` (non-empty) and the list `[0]` (non-empty, its content doesn't matter). Emptiness — not the contained value — decides container truthiness.

## quiz: What does this print?
tags: sorting, stability
track: python

```python
data = [("b", 2), ("a", 2), ("c", 1)]
print(sorted(data, key=lambda p: p[1]))
```

- [ ] `[('c', 1), ('a', 2), ('b', 2)]`
- [x] `[('c', 1), ('b', 2), ('a', 2)]`
- [ ] `[('a', 2), ('b', 2), ('c', 1)]`
- [ ] `[('b', 2), ('a', 2), ('c', 1)]`

> Sorting only by `p[1]` puts the `1` first, then the two `2`s. Python's sort is *stable*, so among equal keys the original order is preserved: `("b", 2)` came before `("a", 2)` in the input, so it stays before it. The first tuple element is never used to break the tie.

## quiz: What does this print?
tags: comprehensions, scope
track: python

```python
x = "outer"
squares = [x for x in range(3)]
print(x)
```

- [ ] `2`
- [x] `outer`
- [ ] raises `NameError`
- [ ] `[0, 1, 2]`

> In Python 3 a comprehension has its own scope, so its loop variable `x` does not leak into the enclosing namespace. The outer `x` is untouched and still `"outer"`. (In Python 2, list comprehensions *did* leak and this would print `2`.)

## quiz: What happens here?
tags: sum, exceptions
track: python

```python
print(sum(["a", "b", "c"]))
```

- [ ] `'abc'`
- [ ] `'cba'`
- [x] raises `TypeError`
- [ ] `0`

> `sum` starts from `0` (an int) and adds each element, so the first step is `0 + "a"` — `int + str` — which raises `TypeError`. Even the string-friendly `sum(seq, "")` is explicitly blocked. To concatenate strings use `"".join(["a", "b", "c"])`.

## quiz: What does this print?
tags: operators, comparison
track: python

```python
x = 10
print(0 < x < 5)
```

- [ ] `True`
- [x] `False`
- [ ] raises `SyntaxError`
- [ ] `10`

> `0 < x < 5` is a *chained* comparison, equivalent to `(0 < x) and (x < 5)`, with `x` evaluated once. That is `True and False`, so `False`. It does **not** mean `(0 < x) < 5`, which would be `True < 5` → `1 < 5` → `True`.

## quiz: What happens when this class is defined?
tags: dataclasses, gotcha
track: python

```python
from dataclasses import dataclass
@dataclass
class Cart:
    items: list = []
```

- [ ] Defines fine; all `Cart` instances share one list
- [x] Raises `ValueError` at class-definition time
- [ ] Defines fine; each `Cart` gets a fresh list
- [ ] Raises `TypeError` only when you first construct a `Cart`

> `@dataclass` detects the mutable default `[]` and refuses it — raising `ValueError: mutable default ... is not allowed: use default_factory` while the class body is being processed (before any instance exists). The fix is `items: list = field(default_factory=list)`.

## quiz: What does this print?
tags: bytes, strings
track: python

```python
data = b"hello"
print(data[0], type(data[0]).__name__)
```

- [ ] `b'h' bytes`
- [x] `104 int`
- [ ] `h str`
- [ ] `104 bytes`

> Indexing a `bytes` object yields an *integer* (the byte value), not a length-1 bytes or a str — `data[0]` is `104`, the ASCII code for `'h'`. To get a single-byte bytes object you'd slice: `data[0:1]` → `b'h'`. This str-vs-bytes asymmetry trips people up: `"hello"[0]` is `'h'`, but `b"hello"[0]` is `104`.

## quiz: What does this print?
tags: booleans, short-circuit
track: python

```python
print(0 or [] or "fallback")
print("" and "x")
```

- [ ] `True` then `False`
- [x] `fallback` then an empty line
- [ ] `[]` then `x`
- [ ] `fallback` then `x`

> `and`/`or` return one of their *operands*, not a bool. `or` returns the first truthy operand: `0` and `[]` are falsy, so it yields `"fallback"`. `and` returns the first falsy operand (short-circuiting): `""` is falsy, so it yields `""`, which prints as a blank line.

## quiz: What does this print?
tags: tuples, syntax
track: python

```python
x = (1)
y = (1,)
print(type(x).__name__, type(y).__name__)
```

- [ ] `tuple tuple`
- [x] `int tuple`
- [ ] `tuple int`
- [ ] `int int`

> Parentheses only group — they do not make a tuple. `(1)` is just the integer `1`. It is the *comma* that builds a tuple, so `(1,)` is a one-element tuple. (Even `1,` without parentheses is a tuple.)

## quiz: What does this print?
tags: slicing, sequences
track: python

```python
s = "abcdef"
print(s[::-1], s[-2:], s[1:-1])
```

- [ ] `abcdef fe abcde`
- [x] `fedcba ef bcde`
- [ ] `fedcba ef bcdef`
- [ ] `fedbca ef bcde`

> `s[::-1]` steps backward over the whole string → `'fedcba'`. `s[-2:]` starts two from the end → `'ef'`. `s[1:-1]` drops the first and last characters (stop is exclusive) → `'bcde'`.

## quiz: What does this print?
tags: booleans, int-subclass
track: python

```python
print(True + True)
print(["zero", "one", "two"][True])
```

- [ ] raises `TypeError`
- [x] `2` then `one`
- [ ] `True` then `zero`
- [ ] `2` then `zero`

> `bool` is a subclass of `int`, with `True == 1` and `False == 0`. So `True + True` is `2`, and indexing with `True` is indexing with `1`, giving the element `"one"`.
