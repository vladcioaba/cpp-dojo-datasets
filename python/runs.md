## quiz: Does string + int run?
tags: types, errors
track: python
difficulty: easy

You concatenate a string literal with an integer variable.

```python
age = 30
print("Age: " + age)
```

- [ ] Prints `Age: 30` — the int is converted automatically
- [x] Raises `TypeError`
- [ ] Prints `Age: age`
- [ ] `SyntaxError` — you cannot add different types

> Python never implicitly converts an `int` to `str` for `+`; the code is syntactically fine but raises `TypeError: can only concatenate str (not "int") to str` at runtime. You must convert explicitly with `str(age)` or use an f-string. This is a deliberate design choice — silent coercion (as in JavaScript) hides bugs.

## quiz: Assigning into a tuple
tags: tuples, errors
track: python
difficulty: easy

You try to replace the first element of a tuple.

```python
t = (1, 2, 3)
t[0] = 99
print(t)
```

- [ ] Prints `(99, 2, 3)`
- [ ] `SyntaxError` — tuples cannot appear on the left of `=`
- [x] Raises `TypeError`
- [ ] Prints `(1, 2, 3)` — the assignment is silently ignored

> Tuples are immutable: they do not implement item assignment, so `t[0] = 99` raises `TypeError: 'tuple' object does not support item assignment` at runtime. It is not a syntax error — the parser accepts it, and the failure only happens when the assignment executes. To "change" a tuple you must build a new one.

## quiz: dict .get vs [] lookup
tags: dicts, errors
track: python
difficulty: easy

Both lines look up a key that is not in the dict.

```python
d = {"a": 1}
print(d.get("b"))
print(d["b"])
```

- [ ] Prints `None` twice
- [ ] Raises `KeyError` on the first line — nothing is printed
- [x] Prints `None`, then raises `KeyError`
- [ ] Prints `None`, then `False`

> `.get(key)` returns `None` (or a supplied default) when the key is missing, so the first line prints `None`. Subscript lookup `d["b"]` is strict and raises `KeyError: 'b'`. Since the first `print` completes before the second line runs, you see output followed by the traceback.

## quiz: Slice past the end vs index past the end
tags: strings, slicing
track: python
difficulty: easy

The string has only three characters; every access goes past the end.

```python
s = "abc"
print(s[1:10])
print(s[10:])
print(s[10])
```

- [ ] Raises `IndexError` on the first line
- [ ] Prints `bc`, then raises `IndexError` on `s[10:]`
- [x] Prints `bc`, then an empty line, then raises `IndexError`
- [ ] Prints `bc`, then an empty line, then `None`

> Slices are clamped to the sequence bounds and never raise: `s[1:10]` gives `"bc"` and `s[10:]` gives `""`. Plain indexing is strict, so `s[10]` raises `IndexError: string index out of range`. Remember: slices are forgiving, indexes are not.

## quiz: / vs // with a negative number
tags: numbers, operators
track: python
difficulty: easy

Compare true division, floor division, and floor division of a negative.

```python
print(7 / 2)
print(7 // 2)
print(-7 // 2)
```

- [ ] `3.5`, `3`, `-3`
- [x] `3.5`, `3`, `-4`
- [ ] `3`, `3`, `-3`
- [ ] `3.5`, `3.5`, `-3.5`

> In Python 3, `/` always produces a float (`3.5`) and `//` floors the result. Flooring rounds toward negative infinity, not toward zero, so `-7 // 2` is `-4` (since -3.5 floors down to -4), not `-3` as C-style truncation would give. This trips up people coming from C or Java.

## quiz: Unpacking three values into two names
tags: unpacking, errors
track: python
difficulty: easy

A three-element list is unpacked into two variables.

```python
data = [1, 2, 3]
a, b = data
print(a, b)
```

- [ ] Prints `1 2` — the extra value is dropped
- [ ] Prints `1 [2, 3]`
- [ ] `SyntaxError`
- [x] Raises `ValueError`

> Tuple/list unpacking requires the number of targets to match the number of items exactly, so this raises `ValueError: too many values to unpack (expected 2, got 3)`. Nothing is silently dropped. To absorb the extras you need a starred target: `a, *b = data` gives `a=1, b=[2, 3]`.

## quiz: Arithmetic on booleans
tags: bool, numbers
track: python
difficulty: easy

Booleans are used directly in arithmetic and comparison.

```python
print(True + True)
print(sum([True, False, True]))
print(True == 1)
```

- [x] `2`, `2`, `True`
- [ ] Raises `TypeError` — you cannot add booleans
- [ ] `TrueTrue`, `2`, `True`
- [ ] `2`, `2`, `False`

> `bool` is a subclass of `int` with `True == 1` and `False == 0`, so `True + True` is `2` and `sum` over booleans counts the `True`s — a common idiom like `sum(x > 0 for x in xs)`. The comparison `True == 1` is genuinely `True`, not just truthy.

## quiz: Shadowing the list builtin
tags: builtins, errors
track: python
difficulty: easy

A variable named `list` is created, then `list(...)` is called.

```python
list = [1, 2, 3]
squares = list(range(3))
print(squares)
```

- [ ] Prints `[0, 1, 2]`
- [ ] `SyntaxError` — `list` is a reserved word
- [x] Raises `TypeError`
- [ ] Raises `NameError`

> `list` is not a keyword, just a name in the builtins scope, so assigning to it is legal and shadows the builtin. The call `list(range(3))` then tries to call the list object `[1, 2, 3]`, raising `TypeError: 'list' object is not callable`. Avoid naming variables `list`, `dict`, `str`, `sum`, `id`, etc.

## quiz: Mutating a list inside a tuple
tags: tuples, mutability
track: python
difficulty: medium

The tuple itself is immutable, but its second element is a list.

```python
t = ("a", [1, 2])
t[1].append(3)
print(t)
```

- [ ] Raises `TypeError` — tuples are immutable
- [x] Prints `('a', [1, 2, 3])`
- [ ] Prints `('a', [1, 2])` — the change does not stick
- [ ] Raises `AttributeError`

> Tuple immutability only means the tuple's slots cannot be rebound — the objects inside can still be mutated. `t[1]` is an ordinary read (allowed), and `.append` mutates the list in place without touching the tuple's structure. So this runs fine and prints `('a', [1, 2, 3])`.

## quiz: Consuming a generator twice
tags: generators, iteration
track: python
difficulty: medium

The same generator object is passed to `list` twice.

```python
g = (x * x for x in range(3))
print(list(g))
print(list(g))
```

- [ ] Prints `[0, 1, 4]` twice
- [ ] Prints `[0, 1, 4]`, then raises `StopIteration`
- [x] Prints `[0, 1, 4]`, then `[]`
- [ ] Raises `TypeError` — a generator can only be iterated once

> Generators are single-use iterators: the first `list(g)` drains it, and internally the generator raises `StopIteration` when done. Iterating an exhausted generator is not an error — `list` catches `StopIteration` and simply produces an empty list. If you need two passes, rebuild the generator or use a list.

## quiz: When does the genexp divide by zero?
tags: generators, errors
track: python
difficulty: medium

A generator expression divides by each number, and the list contains a zero.

```python
nums = [1, 0, 2]
results = (10 // n for n in nums)
print("created")
print(list(results))
```

- [ ] Raises `ZeroDivisionError` on line 2 — nothing is printed
- [ ] Prints `created`, then `[10, 0, 5]`
- [x] Prints `created`, then raises `ZeroDivisionError`
- [ ] Prints `created`, then `[10, 5]` — the bad value is skipped

> Generator expressions are lazy: creating one does no division at all, so line 2 succeeds and `created` prints. The `ZeroDivisionError` only fires when `list(results)` pulls the second item and evaluates `10 // 0`. Lazy evaluation moves errors from creation time to consumption time — a classic debugging trap.

## quiz: Read before assignment inside a function
tags: scope, errors
track: python
difficulty: medium

The function reads `count` before assigning to it later in the body.

```python
count = 0
def bump():
    print(count)
    count = 1
bump()
```

- [ ] Prints `0` — the global is visible until the assignment
- [ ] Prints `0`, and the global becomes `1`
- [x] Raises `UnboundLocalError`
- [ ] Raises `NameError: name 'count' is not defined`

> Scope is decided at compile time for the whole function: because `count = 1` appears anywhere in the body, `count` is local everywhere in `bump`. The `print` therefore reads a local that has no value yet, raising `UnboundLocalError` (a subclass of `NameError`, but reported distinctly). Declaring `global count` would make it print `0`.

## quiz: Does the comprehension variable leak?
tags: scope, comprehensions
track: python
difficulty: medium

A list comprehension reuses the name `x` that already exists outside.

```python
x = "outer"
squares = [x * x for x in range(3)]
print(x)
```

- [x] Prints `outer`
- [ ] Prints `2` — the loop variable leaks out
- [ ] Prints `4` — the last computed square
- [ ] Raises `NameError`

> In Python 3 a comprehension runs in its own scope, so its loop variable `x` is completely separate from the outer `x` and does not leak. The outer binding is untouched and `outer` prints. (In Python 2 list comprehensions did leak, which is exactly why this was changed.) A plain `for` loop, by contrast, still leaks its variable.

## quiz: Where does the walrus variable live?
tags: walrus, scope
track: python
difficulty: medium

`n` is bound with `:=` inside the `if` condition, then used after the block.

```python
data = [4, 8, 15]
if (n := len(data)) > 2:
    print("big")
print(n)
```

- [ ] Prints `big`, then raises `NameError` — `n` only exists in the condition
- [x] Prints `big`, then `3`
- [ ] `SyntaxError` — `:=` is not allowed inside `if`
- [ ] Prints `big`, then `True`

> The walrus operator `:=` (Python 3.8+) assigns and yields the value in one expression, and the name goes into the enclosing function/module scope — Python has no block scope. So `n` is `3`, the condition is true, and `n` remains visible after the `if`. Note it binds the length, not the comparison result.

## quiz: Chained comparison with in
tags: comparisons, gotcha
track: python
difficulty: medium

This looks like it checks that membership is `True`.

```python
a = [1]
print(1 in a == True)
```

- [ ] Prints `True`
- [x] Prints `False`
- [ ] `SyntaxError` — `in` and `==` cannot be mixed
- [ ] Raises `TypeError`

> `in` and `==` are both comparison operators, so this chains: `1 in a == True` means `(1 in a) and (a == True)`. The first part is `True`, but `[1] == True` is `False`, so the whole thing prints `False`. Chaining is great for `0 < x < 10` but silently produces nonsense when you mix membership and equality — parenthesize `(1 in a) == True`, or just drop the comparison.

## quiz: Lambdas created in a loop
tags: closures, gotcha
track: python
difficulty: medium

Three lambdas capture the comprehension variable `i`.

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])
```

- [ ] Prints `[0, 1, 2]`
- [x] Prints `[2, 2, 2]`
- [ ] Raises `NameError` — `i` is gone when the lambdas run
- [ ] Prints `[3, 3, 3]`

> Closures capture the variable, not its value at creation time. All three lambdas share the same `i` cell, which holds `2` after the loop finishes, so each call returns `2`. The variable outlives the comprehension inside the closure cells, so there is no `NameError`. The standard fix is a default argument: `lambda i=i: i`, which snapshots the value.

## quiz: Sorting a mixed list
tags: sorting, errors
track: python
difficulty: medium

The list mixes ints and a string, then gets sorted.

```python
items = [3, "1", 2]
items.sort()
print(items)
```

- [ ] Prints `["1", 2, 3]` — strings sort first
- [ ] Prints `[2, 3, "1"]` — ints sort first
- [x] Raises `TypeError`
- [ ] Prints `[1, 2, 3]` — the string is converted

> Sorting compares elements pairwise with `<`, and Python 3 refuses to order unrelated types: `'1' < 3` raises `TypeError: '<' not supported between instances of 'str' and 'int'`. (Python 2 allowed this with an arbitrary but consistent order.) To sort mixed data you must supply a key, e.g. `items.sort(key=int)` or `key=str`.

## quiz: When is an f-string evaluated?
tags: f-strings, evaluation
track: python
difficulty: medium

The variable changes after the f-string is created but before it is printed.

```python
x = 1
msg = f"x is {x}"
x = 2
print(msg)
```

- [x] Prints `x is 1`
- [ ] Prints `x is 2` — f-strings resolve when printed
- [ ] Prints `x is {x}`
- [ ] Raises `NameError`

> An f-string is an expression evaluated immediately where it appears — it interpolates `x` at assignment time, producing the plain string `"x is 1"`. Rebinding `x` afterwards cannot affect a string that already exists. F-strings are not lazy templates; for deferred substitution you would use `str.format` on a template string later.

## quiz: else on a for loop
tags: loops, else
track: python
difficulty: medium

A search loop over odd numbers has an `else` clause.

```python
for n in [1, 3, 5]:
    if n % 2 == 0:
        print("found even")
        break
else:
    print("no even found")
```

- [ ] `SyntaxError` — `else` cannot attach to `for`
- [ ] Prints nothing — the `else` belongs to the `if`, which is never true
- [x] Prints `no even found`
- [ ] Prints `no even found` three times, once per iteration

> A `for` loop can have an `else` clause that runs only if the loop finishes without hitting `break`. No element here is even, so `break` never fires and the `else` body runs exactly once after the loop. Think of it as "no-break". Indentation makes it the loop's `else`, not the `if`'s.

## quiz: Deleting dict keys while iterating
tags: dicts, errors
track: python
difficulty: medium

A key is deleted from the dict inside a loop over that same dict.

```python
d = {"a": 1, "b": 2}
for k in d:
    if d[k] == 1:
        del d[k]
print(d)
```

- [ ] Prints `{'b': 2}`
- [ ] Prints `{'a': 1, 'b': 2}` — the deletion is deferred
- [ ] Raises `KeyError`
- [x] Raises `RuntimeError`

> Adding or removing keys while iterating over a dict invalidates the iterator, and Python detects it on the next step: `RuntimeError: dictionary changed size during iteration`. Overwriting values of existing keys is fine; changing the key set is not. The idiomatic fix is to iterate over a snapshot — `for k in list(d):` — or build a new dict comprehension.

## quiz: += on a list stored in a tuple
tags: tuples, gotcha
track: python
difficulty: hard

Augmented assignment targets a tuple slot that holds a list.

```python
t = ([1, 2],)
try:
    t[0] += [3]
except TypeError:
    print("TypeError")
print(t)
```

- [ ] Prints `([1, 2, 3],)` only — no exception, `+=` mutates the list
- [ ] Prints `TypeError` then `([1, 2],)` — the exception prevents any change
- [x] Prints `TypeError` then `([1, 2, 3],)` — it raises AND the list changed
- [ ] Raises `SyntaxError`

> `t[0] += [3]` executes in two steps: first `list.__iadd__` extends the list in place (this succeeds — the list is now `[1, 2, 3]`), then Python tries to store the result back with `t[0] = ...`, which fails because tuples reject item assignment. So the `TypeError` is raised after the mutation already happened. It is Python's most famous "fails and succeeds at the same time" snippet.

## quiz: global declared inside a nested function
tags: scope, global
track: python
difficulty: hard

`inner` declares `global x` while an enclosing `outer` also has its own `x`.

```python
x = "global"
def outer():
    x = "outer"
    def inner():
        global x
        x = "changed"
    inner()
    print(x)
outer()
print(x)
```

- [ ] Prints `changed` then `changed`
- [x] Prints `outer` then `changed`
- [ ] Prints `outer` then `global`
- [ ] Raises `SyntaxError` — `global` is not allowed in a nested function

> `global` skips all enclosing function scopes and binds directly to the module level, so `inner` rewrites the module's `x` and leaves `outer`'s local `x` untouched — hence `outer` prints first, then `changed`. To rebind the enclosing function's variable instead, `inner` would need `nonlocal x`, which would print `changed` then `global`.

## quiz: try/except/else/finally order
tags: exceptions, control-flow
track: python
difficulty: hard

The same function runs once without and once with an exception.

```python
def check(n):
    try:
        10 // n
    except ZeroDivisionError:
        print("except")
    else:
        print("else")
    finally:
        print("finally")
check(2)
check(0)
```

- [ ] `else`, `finally`, `except`, `else`, `finally`
- [ ] `else`, `except`, `finally`, `finally`
- [x] `else`, `finally`, `except`, `finally`
- [ ] `finally`, `else`, `finally`, `except`

> The `else` block runs only when the `try` body raised nothing, and it is skipped entirely when an exception was caught — `else` and `except` are mutually exclusive. `finally` runs unconditionally, last, in both cases. So `check(2)` prints `else`, `finally` and `check(0)` prints `except`, `finally`.

## quiz: Infinite recursion — hang or crash?
tags: recursion, errors
track: python
difficulty: hard

The function calls itself unconditionally with no base case.

```python
def depth(n):
    return depth(n + 1)
print(depth(0))
```

- [ ] Hangs forever — you must kill the process
- [ ] Crashes the interpreter with a segmentation fault
- [x] Raises `RecursionError`
- [ ] Raises `StackOverflowError`

> CPython guards its call stack with a recursion limit (about 1000 by default, see `sys.getrecursionlimit()`), and blowing past it raises `RecursionError: maximum recursion depth exceeded` — a catchable Python exception, not a hang or a hard crash. `StackOverflowError` is Java, not Python. Python also does no tail-call optimization, so rewriting this as a "tail call" would not help.

## quiz: Do two calls share the default list?
tags: functions, gotcha
track: python
difficulty: hard

Two separate calls each rely on the default `bucket`, then identity is checked.

```python
def collect(x, bucket=[]):
    bucket.append(x)
    return bucket
a = collect(1)
b = collect(2)
print(a is b, a)
```

- [ ] Prints `False [1]` — each call gets a fresh list
- [ ] Prints `True [2]` — the second call replaces the contents
- [x] Prints `True [1, 2]`
- [ ] Raises `TypeError` — mutable defaults are not allowed

> Default values are evaluated once, at `def` time, and stored on the function object — every call that omits `bucket` gets the same list. Both calls append to that one object, so `a` and `b` are the same list (`a is b` is `True`) containing `[1, 2]`. The idiom to get a fresh list per call is `bucket=None` plus `if bucket is None: bucket = []`.
