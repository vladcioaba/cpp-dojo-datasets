## fact: CPython compiles your code too — just not to machine code
tags: toolchain, interpreter, bytecode
track: python

"Interpreted" undersells what CPython does. Your source goes through a real compiler pipeline: **tokenizer** (text → tokens like `NAME 'price'`, `OP '*'`), **parser** (tokens → AST, via a PEG grammar since 3.9), **compiler** (AST → **bytecode**, instructions for CPython's stack-based virtual machine). Only then does interpretation start: the **ceval loop** — a giant dispatch loop in C — fetches each bytecode instruction and executes its handler.

The difference from C++ is *where the pipeline stops*. g++ continues down to native machine code ahead of time; CPython stops at portable bytecode and pays for the rest at runtime, one dispatch per instruction. That late binding is also what buys Python its dynamism — the meaning of `a + b` is decided each time the instruction runs.

```python
import ast
print(ast.dump(ast.parse("total = price * 2")))
# Module(body=[Assign(targets=[Name(id='total', ctx=Store())],
#   value=BinOp(left=Name(id='price', ctx=Load()), op=Mult(),
#               right=Constant(value=2)))])
```

## fact: dis shows you the bytecode your function actually runs
tags: toolchain, interpreter, dis
track: python

The `dis` module disassembles any function into the VM instructions the ceval loop will execute — the Python equivalent of reading `-S` assembly output. The VM is stack-based: operands are pushed, operations pop them and push results. `LOAD_FAST` pushes a local variable, `LOAD_CONST` pushes a constant, `BINARY_OP` pops two and pushes the result, `STORE_FAST` pops into a local, `RETURN_VALUE` returns the top of stack.

Reading `dis` output answers real questions: why local variable access beats globals (`LOAD_FAST` is an array index; `LOAD_GLOBAL` is a dict lookup), what a comprehension actually costs, and what CPython's specializing interpreter (3.11+) rewrites your instructions into. Exact opcode names shift between versions — 3.14 below emits `LOAD_FAST_BORROW`, a specialized `LOAD_FAST` — but the shape is stable.

```python
import dis
def greet(name):
    msg = 'hello ' + name
    return msg
dis.dis(greet)
#  4    LOAD_CONST               0 ('hello ')
#       LOAD_FAST_BORROW         0 (name)
#       BINARY_OP                0 (+)
#       STORE_FAST               1 (msg)
#  5    LOAD_FAST_BORROW         1 (msg)
#       RETURN_VALUE
```

## fact: .pyc files cache compilation, not execution
tags: toolchain, interpreter, pyc
track: python

Compiling source to bytecode takes time, so CPython caches the result: importing `mymod.py` writes `__pycache__/mymod.cpython-314.pyc` — the marshaled code object, tagged with the interpreter version so different Pythons coexist. The payoff is *import* speed only; the bytecode runs at exactly the same speed either way. (Note: only imported modules get cached — the script you run directly is compiled fresh each time.)

Staleness is handled by a 16-byte header: a **magic number** (bytecode format version — any pyc from another version is ignored wholesale), flags, and the source file's **mtime and size**. On import, CPython compares the recorded mtime/size against the current `.py`; any mismatch triggers silent recompilation. PEP 552 added an alternative **hash-based** mode (flags bit set, SipHash of the source instead of mtime) for reproducible builds and build systems that don't preserve timestamps. Practical corollary: deleting `__pycache__` is never dangerous, just slightly slow.

```python
import struct
raw = open('__pycache__/mymod.cpython-314.pyc', 'rb').read(16)
magic, flags, mtime, size = struct.unpack('<4sIII', raw)
# flags == 0 -> timestamp-based: stale if mtime/size differ from mymod.py
```

## fact: The GIL serializes bytecode, not your whole program
tags: toolchain, interpreter, gil
track: python

The Global Interpreter Lock is a single mutex a thread must hold to *execute Python bytecode*. It exists because CPython's memory management — reference counts mutated on nearly every operation — isn't thread-safe on its own; one lock around the interpreter is cheaper than fine-grained locking everywhere. The consequence: within one process, only one thread runs Python code at a time. Four CPU-bound threads take roughly as long as one (often slightly longer, from lock contention).

Switching is time-based: a running thread checks for a drop request at bytecode boundaries, and a waiting thread triggers one every 5 ms by default (`sys.getswitchinterval()`). Crucially, the GIL is *released* around blocking I/O and inside many C extensions — so threads genuinely help I/O-bound work, and NumPy-heavy code can use cores. For CPU-bound pure Python, the classic answer is `multiprocessing` (one GIL per process); since 3.13 there's also an experimental **free-threaded** build with the GIL removed entirely.

```python
import sys
sys.getswitchinterval()   # 0.005 — a waiting thread requests the GIL
                          # after 5ms; holder yields between bytecodes
```

## fact: CPython is slow where a JIT would be fast — and that's a design choice
tags: toolchain, interpreter, jit
track: python

Two taxes dominate. **Dynamic lookup**: `obj.method()` can't be resolved ahead of time — every call walks the instance dict, then the class and its MRO, because anything may have been monkey-patched between calls. C++ resolves the same call to a fixed offset at compile time. **Boxing**: there are no raw machine ints; `1 + 2` operates on heap-allocated `PyLong` objects with type checks and refcount traffic per operation, which is also why a million-element Python list of ints is many times larger than a C array.

A **JIT** (just-in-time compiler) attacks both: watch the running program, observe that `x` in this loop is always an int, compile a specialized native-code version with unboxed arithmetic, and guard it (deoptimize if the assumption breaks). **PyPy** does exactly this and runs pure-Python loops 5–50x faster. CPython is converging on the same playbook in-tree: 3.11's specializing adaptive interpreter rewrites hot bytecodes based on observed types, and 3.13 added an experimental copy-and-patch JIT.

```python
x = 0
for i in range(1_000_000):
    x += i        # CPython: boxed PyLongs, per-iteration dispatch
                  # PyPy JIT: observed int -> native add in a register
```

## fact: Fast Python is Python that hands the loop to C
tags: toolchain, interpreter, c-extensions
track: python

CPython's escape hatch is the **C API**: a compiled extension module looks like a normal import but its functions run native code. This is the NumPy pattern — `a + b` on two arrays executes *one* bytecode dispatch, then a compiled C loop processes a million elements with unboxed doubles in contiguous memory. The interpreter overhead is amortized across the whole array instead of paid per element. Long-running C work can also release the GIL, so it parallelizes across threads.

The corollary is the golden rule of numeric Python: **don't write the loop yourself**. A Python-level `for` over a NumPy array is the worst of both worlds — per-element dispatch *plus* boxing each scalar out of the array. Vectorize, or move the kernel to C/C++ yourself via Cython, pybind11, or the raw C API: the same escape hatch NumPy uses is how C++ libraries grow Python bindings.

```python
import numpy as np
a = np.arange(1_000_000)
b = a * 2                 # one dispatch, C loop, GIL-friendly
c = [x * 2 for x in a]    # ~million dispatches, boxes every element
```
