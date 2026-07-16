## quiz: Both files compile, the build still fails — which stage, and why?
tags: toolchain, compiler, linker
track: core

`legacy.c` (compiled as C) defines `int checksum(const char* s)`. Your C++ code declares it in a header and calls it. Both files compile cleanly, but the build fails:

```text
undefined reference to `checksum(char const*)'
```

- [ ] The preprocessor — the header declaring `checksum` was not found
- [ ] The compiler front end — the declaration's types don't match the definition
- [x] The linker — C++ looks for a mangled symbol like `_Z8checksumPKc`, but the C object exports plain `checksum`; declare it `extern "C"`
- [ ] The assembler — C and C++ produce incompatible object file formats

> Each stage owns an error class: a missing header dies in the preprocessor, a missing semicolon in the compiler front end, an unresolved symbol only at link time. C++ mangles names — encoding parameter types into the symbol so overloads can coexist — so the C++ caller wants `_Z8checksumPKc` while the C object defines `checksum`. Wrapping the declaration in `extern "C"` disables mangling and makes both sides agree. Object formats are identical for C and C++; the assembler is not involved.

## quiz: What does #pragma once actually prevent?
tags: toolchain, compiler, preprocessor
track: core

- [ ] The same symbol being defined in two different .cpp files (a link error)
- [x] The same header's contents being pasted twice into one translation unit
- [ ] Other headers from re-declaring names this header declares
- [ ] ODR violations across the whole program

> `#include` is textual paste; without protection, a header reached twice (directly and via another header) would define its classes twice *in that one TU* — an immediate compile error. `#pragma once`, like classic include guards, makes the second inclusion expand to nothing. Its scope ends at the translation unit boundary: it does nothing about two .cpp files both defining the same symbol, and nothing for cross-TU ODR discipline.

## quiz: Two TUs saw different versions of an inline function — what happens?
tags: toolchain, compiler, odr
track: core

`a.cpp` was compiled against yesterday's `price.h`; `b.cpp` against today's, in which the inline function `price_ticks()` changed. You link both object files together.

- [ ] The linker reports a multiple definition error
- [ ] The linker reports an ODR violation and picks the newer definition
- [x] It links cleanly; the linker silently keeps one copy, and the program has undefined behavior
- [ ] Each TU keeps calling its own version, which is well-defined

> Inline functions emit a weak (COMDAT) copy in every TU that uses them, and the linker's job is to deduplicate — it assumes the copies are identical, as the ODR requires, and does not compare them. One definition survives; the TU built against the other now runs code disagreeing with what it was compiled against. That's undefined behavior with the classic signature: links fine, works in Debug, misbehaves in Release or when link order changes.

## quiz: Static vs dynamic linking — which tradeoff is stated correctly?
tags: toolchain, compiler, linking
track: core

- [ ] Dynamic linking copies the library into the executable, so the binary is larger but self-contained
- [ ] Static linking lets you patch a shared library and fix every already-deployed program without relinking
- [x] Static linking yields a self-contained binary at the cost of size and needing a relink to pick up library fixes; dynamic linking shares and updates libraries independently but can fail at load time with a missing or wrong-version .so
- [ ] The two produce identical deployment behavior; the difference is only build speed

> Static linking (`libfoo.a`) copies the needed library code into your executable: nothing to install on the target machine, but every binary carries its own copy and a library fix means relink-and-redeploy. Dynamic linking (`libfoo.so`) records a dependency resolved at load time: one shared copy, independently patchable — and a new deployment failure mode when the library is absent or ABI-incompatible. The first two options describe each approach with the other's properties.

## quiz: Your benchmark loop reports 0 ns at -O2 — what happened?
tags: toolchain, compiler, optimization
track: core

```cpp
auto t0 = clock::now();
long sum = 0;
for (int i = 0; i < 1'000'000; ++i)
    sum += i;                     // sum never used afterwards
auto t1 = clock::now();           // t1 - t0 ~ 0ns at -O2
```

- [ ] The CPU executed the loop speculatively before `t0` was taken
- [ ] -O2 vectorized the loop, making it too fast to measure
- [x] The result is unobservable, so the compiler deleted the loop (dead code elimination); it may also have computed `sum` at compile time
- [ ] `clock::now()` is only precise to milliseconds

> Optimizers operate under the as-if rule: any transformation preserving observable behavior is legal. A `sum` that nothing reads has no observable effect, so -O2 removes the loop entirely — and even if `sum` were read, a loop over constants can be folded to a single precomputed value. Vectorization would still leave microseconds of measurable work. Benchmarks must force observability: use the result, or `benchmark::DoNotOptimize(sum)`.

## quiz: A zeroed 4 MB array vs an initialized one — where do the bytes live?
tags: toolchain, compiler, object-files
track: core

```cpp
int zeros[1'000'000];              // global, zero-initialized
int ones[1'000'000] = {1, 1, 1};   // global, explicitly initialized
```

- [ ] Both add ~4 MB each to the executable file
- [x] `zeros` goes in .bss (recorded as just a size, no file bytes); `ones` goes in .data and stores ~4 MB in the file
- [ ] Both are stored compressed in .rodata
- [ ] `zeros` is placed in .text so the loader can generate it

> .data holds globals with nonzero initial values and must store them byte-for-byte in the binary. .bss holds zero-initialized (and uninitialized) globals as metadata only — a size — and the loader zero-fills that region at startup for free. So `zeros` costs nothing on disk while `ones` costs the full 4 MB (a nonzero element anywhere disqualifies it from .bss). .rodata is for constants like string literals; .text is code.

## quiz: app can't find json.hpp, but core links fine — what's the right fix?
tags: toolchain, cmake, propagation
track: core

`core.h` (a public header of `core`) does `#include <nlohmann/json.hpp>`. The project has:

```cmake
target_link_libraries(core PRIVATE nlohmann_json::nlohmann_json)
target_link_libraries(app PRIVATE core)
# app's compile of main.cpp fails: 'nlohmann/json.hpp' file not found
```

- [ ] Add the JSON include directory to `app` with `target_include_directories`
- [x] Change the scope: `target_link_libraries(core PUBLIC nlohmann_json::nlohmann_json)`
- [ ] Also link json into app: `target_link_libraries(app PRIVATE nlohmann_json::nlohmann_json)`
- [ ] Change `app`'s link to `INTERFACE`: `target_link_libraries(app INTERFACE core)`

> PRIVATE declares "only core's implementation uses this" — so the dependency's usage requirements (its include dirs) don't propagate to core's consumers. But core's *public header* mentions json, meaning every consumer needs it: that is precisely PUBLIC. Hand-adding include paths to app, or making app link json directly, patches this one consumer and breaks again for the next; it also lies about the dependency graph. The rule: appears in your public headers → PUBLIC; implementation-only → PRIVATE.

## quiz: find_package vs FetchContent — which statement is true?
tags: toolchain, cmake, dependencies
track: core

- [ ] find_package downloads the library's source at configure time; FetchContent locates an installed copy
- [x] find_package consumes a dependency already installed on the system; FetchContent downloads the source at configure time and builds it inside your build tree
- [ ] FetchContent can only fetch header-only libraries
- [ ] find_package and FetchContent cannot produce the same imported targets

> find_package locates something provisioned beforehand (system package, vcpkg/conan, manual install) via its Config file or a Find module — fast builds, but the machine must be set up. FetchContent removes the provisioning step: it downloads a pinned tag and `add_subdirectory`s it, so a clean clone builds with zero system state, at the cost of compiling the dependency yourself. Any library with well-behaved CMake works, not just header-only — and both routes typically expose identical targets like `fmt::fmt`, which is what makes hybrid setups (FIND_PACKAGE_ARGS fallback) seamless.

## quiz: A pointer into a vector goes bad after push_back — which sanitizer?
tags: toolchain, sanitizers
track: core

Your tests intermittently read garbage through a `Order*` that points into a `std::vector<Order>`; you suspect a reallocation freed the old buffer. Which tool reports this directly, with the stack traces of the bad read, the free, and the original allocation?

- [x] AddressSanitizer (-fsanitize=address)
- [ ] ThreadSanitizer (-fsanitize=thread)
- [ ] UndefinedBehaviorSanitizer (-fsanitize=undefined)
- [ ] MemorySanitizer (-fsanitize=memory)

> Reading through a pointer into a freed heap block is heap-use-after-free — ASan's home turf. It quarantines freed memory and poisons red zones, so the bad read aborts immediately with three stacks: where you read, where the buffer was freed (the reallocation), and where it was allocated. TSan finds races between threads; UBSan finds language-rule violations like signed overflow; MSan finds reads of *uninitialized* (never-written) memory, which is a different bug than use-after-free.

## quiz: Two threads increment a counter and the total comes up short — which sanitizer?
tags: toolchain, sanitizers
track: core

A stats counter incremented from two threads without synchronization occasionally ends below the expected total. It never crashes. Which tool identifies the root cause, naming both racing stacks?

- [ ] AddressSanitizer (-fsanitize=address)
- [ ] UndefinedBehaviorSanitizer (-fsanitize=undefined)
- [x] ThreadSanitizer (-fsanitize=thread)
- [ ] MemorySanitizer (-fsanitize=memory)

> Unsynchronized read-modify-write from two threads is a data race — undefined behavior even when it "only" drops updates. TSan instruments every memory access, tracks the happens-before graph, and reports the two conflicting stacks plus the threads' identities, catching races even on runs where the corruption didn't happen to manifest. ASan sees no invalid address here (the counter is valid memory), and UBSan's checks don't include cross-thread synchronization. Remember TSan runs alone: it cannot be combined with ASan or MSan, and expect 5-15x slowdown.

## quiz: Tick math goes weird only in release builds at large values — which sanitizer?
tags: toolchain, sanitizers
track: core

`int position_value = price_ticks * quantity;` produces bizarre negative numbers in production for large positions, but debug builds "work". Which tool pinpoints the exact line and values?

- [ ] AddressSanitizer (-fsanitize=address)
- [ ] ThreadSanitizer (-fsanitize=thread)
- [x] UndefinedBehaviorSanitizer (-fsanitize=undefined)
- [ ] MemorySanitizer (-fsanitize=memory)

> Signed integer overflow is undefined behavior — the optimizer may assume it never happens, which is exactly why symptoms appear at -O2 and not in debug builds. UBSan instruments the arithmetic and prints `runtime error: signed integer overflow: 2000000000 * 2 cannot be represented in type 'int'` with file and line. No invalid memory is touched, so ASan is silent; no threads are involved, so TSan is irrelevant; the operands are fully initialized, so MSan has nothing to say. UBSan is cheap enough to leave on in every test build.

## quiz: Which Python function produced this disassembly?
tags: toolchain, interpreter, dis
track: python

```text
LOAD_CONST               0 ('hello ')
LOAD_FAST_BORROW         0 (name)
BINARY_OP                0 (+)
STORE_FAST               1 (msg)
LOAD_FAST_BORROW         1 (msg)
RETURN_VALUE
```

- [ ] `def f(name): return 'hello ' + name`
- [x] `def f(name): msg = 'hello ' + name; return msg`
- [ ] `def f(name): return GREETING + name`
- [ ] `def f(name): return f'hello {name}'`

> Read it as stack traffic: push the constant `'hello '`, push the parameter (`LOAD_FAST_BORROW` is 3.14's variant of `LOAD_FAST` — locals are indexed slots), `BINARY_OP (+)` pops both and pushes the result. The giveaway is `STORE_FAST`/`LOAD_FAST` on `msg`: the result lands in a local and is loaded back — so a temporary variable exists. Returning the expression directly would go straight to `RETURN_VALUE`; a module-level `GREETING` would use `LOAD_GLOBAL`; an f-string compiles to `FORMAT_SIMPLE`/`BUILD_STRING` instructions, not `BINARY_OP`.

## quiz: You edit mymod.py while a stale .pyc sits in __pycache__ — what runs?
tags: toolchain, interpreter, pyc
track: python

`__pycache__/mymod.cpython-314.pyc` was compiled from yesterday's source. You save changes to `mymod.py` and run `python -c "import mymod"`.

- [ ] The stale bytecode runs; you must delete __pycache__ to pick up edits
- [ ] Python raises ImportError because the cache and source disagree
- [x] Python compares the source's mtime and size against values stored in the .pyc header, sees a mismatch, recompiles, and rewrites the cache
- [ ] The .pyc is only refreshed when the CPython version changes

> The 16-byte .pyc header stores a magic number (bytecode format version) plus the source's mtime and size (or, in PEP 552 hash-based mode, a hash of the source). On import, a mismatch triggers silent recompilation and the cache is rewritten — so stale-bytecode bugs essentially don't happen through normal edits. The magic number handles the version case: a .pyc from a different CPython is ignored entirely, which is why the interpreter version is baked into the filename. Deleting __pycache__ is always safe, merely slower.

## quiz: 8 seconds of pure-Python math, 4 threads, 8 cores — how long?
tags: toolchain, interpreter, gil
track: python

A CPU-bound pure-Python function takes 8s single-threaded. You split the work evenly across 4 `threading.Thread`s on an 8-core machine (standard CPython). Expected wall time?

- [ ] ~2s — four threads on eight cores
- [ ] ~4s — the GIL allows two threads to run concurrently
- [x] ~8s, possibly slightly worse than single-threaded
- [ ] ~2s, but only if the threads are created before the work starts

> The GIL is one mutex that a thread must hold to execute Python bytecode, so CPU-bound pure-Python threads run strictly one at a time — the work is serialized regardless of core count, and GIL handoffs every 5ms (`sys.getswitchinterval()`) plus cache churn often make it slightly *slower* than one thread. Threads do help when the GIL is released: blocking I/O and long-running C-extension work (NumPy kernels). For CPU-bound pure Python, use `multiprocessing` — one interpreter and GIL per process — or the experimental free-threaded build (3.13+).
