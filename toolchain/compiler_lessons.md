## fact: Four stages stand between your .cpp and a running binary
tags: toolchain, compiler, pipeline
track: core

`g++ main.cpp` looks like one step but runs four: **preprocess** (expand `#include` and macros — pure text), **compile** (C++ → assembly), **assemble** (assembly → object file), **link** (combine object files and libraries into an executable). Each stage has its own failure mode: a missing header dies in the preprocessor, a syntax error in the compiler, an unresolved symbol only at link time.

You can stop the driver at any stage — this is the fastest way to build a mental model of what each one produces.

```bash
g++ -E main.cpp -o main.ii   # preprocess only: one giant translation unit
g++ -S main.ii  -o main.s    # compile only: human-readable assembly
g++ -c main.s   -o main.o    # assemble only: relocatable object file
g++ main.o      -o main      # link: resolve symbols, produce executable
```

## fact: The preprocessor is a text machine that has never heard of C++
tags: toolchain, compiler, preprocessor
track: core

Before compilation proper, the preprocessor runs a purely textual pass: `#include "x.h"` is literally replaced by the file's contents (recursively — a hello-world TU can balloon to tens of thousands of lines), and `#define` macros are token substitutions with no knowledge of types or scopes. That ignorance is why function-like macros need parenthesized arguments: `SQUARE(a+b)` expands to whatever text you wrote.

Because inclusion is textual, a header pulled in twice would define its contents twice. Include guards (`#ifndef FOO_H / #define FOO_H / #endif`) make the second inclusion expand to nothing. `#pragma once` does the same job keyed on file identity — less typing, no name-collision risk, supported by every major compiler, but technically non-standard. Both only guard *within one translation unit*; they do nothing about the same symbol defined in different .cpp files.

```cpp
// widget.h
#pragma once            // or the classic guard:
#ifndef WIDGET_H
#define WIDGET_H
struct Widget { int id; };
#endif
```

## fact: Lexing and parsing turn text into a tree the compiler can reason about
tags: toolchain, compiler, parsing
track: core

The compiler front end first **lexes** the preprocessed text into tokens (`int`, `x`, `=`, `42`, `;`), then **parses** the token stream against the language grammar into an **AST** — an abstract syntax tree where `a + b * c` becomes a `+` node whose right child is a `*` node. Precedence, associativity, and matching braces live here; a missing semicolon or unbalanced parenthesis is a *parse* error.

C++ is notoriously hard to parse because syntax alone is ambiguous: `x * y;` is a multiplication or a pointer declaration depending on whether `x` names a type. So the C++ parser can't be a pure grammar machine — it consults the symbol table while parsing (the "lexer hack" family of problems, and why `typename` is sometimes required in templates).

```cpp
a + b * c
//    (+)          AST: precedence is structure,
//   /   \         not left-to-right text order
//  a    (*)
//      /   \
//     b     c
```

## fact: Semantic analysis is where templates finally become code
tags: toolchain, compiler, templates
track: core

After parsing, **semantic analysis** walks the AST checking meaning: name lookup, type checking, overload resolution, implicit conversions, access control. "Cannot convert `const char*` to `int`" is a semantic error — the syntax was fine.

Templates get special treatment: a template definition is only lightly checked when declared; the real work happens at the **point of instantiation**, when you use `std::vector<MyType>` with concrete arguments. Only then does the compiler substitute the types, re-run lookup and type checking, and generate an actual class or function. This is why template errors explode with page-long instantiation stacks, why a broken template can sit uncompiled in a header for months until someone instantiates it — and why C++20 concepts help, by checking requirements at the call site before instantiation dives in.

```cpp
template <class T>
T max_of(T a, T b) { return a < b ? a : b; }   // checked shallowly here

max_of(1, 2);        // instantiation point: max_of<int> generated now
max_of(p1, p2);      // error appears HERE if P has no operator<
```

## fact: Optimization happens on IR, and -O2 is allowed to delete your code
tags: toolchain, compiler, optimization
track: core

Compilers don't optimize your source or the final assembly — they lower the AST to an **intermediate representation** (LLVM IR, GCC GIMPLE) and run dozens of passes over it: **inlining** (paste the callee's body, enabling further passes), **constant folding** (`2 * 3600` becomes `7200` at compile time), **dead code elimination** (anything provably unobservable is removed), **vectorization** (rewrite a loop to process 4–16 elements per SIMD instruction).

The flags are ratchet levels: `-O0` means no optimization, fast compiles, debuggable line-by-line — and is what you get by default. `-O2` enables the standard battery and is the production baseline. `-O3` adds aggressive loop transforms (more vectorization and unrolling) — often faster, sometimes bigger and occasionally slower. The classic trap: a benchmark loop whose result is never used is *dead code*, and `-O2` will happily delete it and report zero nanoseconds.

```cpp
for (int i = 0; i < 1'000'000; ++i)
    sum += data[i];          // result unused afterwards?
// -O2: entire loop removed (DCE). Benchmark says 0ns.
// Fix: use the result, e.g. benchmark::DoNotOptimize(sum).
```

## fact: Code generation is a fight over sixteen registers
tags: toolchain, compiler, codegen
track: core

The back end lowers optimized IR to real machine instructions: **instruction selection** (pick concrete opcodes for each IR operation), **scheduling** (order them to keep CPU pipelines busy), and **register allocation** — the hard one. Your function may juggle fifty live values; x86-64 gives roughly sixteen general-purpose registers. The allocator (graph coloring or linear scan) decides who gets a register and who gets **spilled** to a stack slot, adding memory traffic that never existed in your source.

The back end also implements the platform **calling convention** (System V on Linux/macOS: first integer args in `rdi, rsi, rdx, rcx, r8, r9`, return in `rax`) — the contract that lets separately compiled functions call each other. Hot-loop performance often comes down to whether the allocator kept your working set in registers; too many live variables in a loop body is a real, measurable cost.

```bash
g++ -O2 -S hot.cpp -o hot.s      # read the generated assembly
grep -c "rsp" hot.s              # spill traffic touches the stack
```

## fact: An object file is compiled code filed into named sections
tags: toolchain, compiler, object-files
track: core

The assembler produces a **relocatable object file** (`.o`, ELF on Linux) — machine code plus bookkeeping, organized into sections: **.text** (the code, read-only + executable), **.rodata** (constants and string literals), **.data** (globals initialized to nonzero values — stored byte-for-byte in the file), **.bss** (zero/uninitialized globals — recorded as *just a size*, occupying no file space; the loader zero-fills it at startup). A 100 MB zeroed array costs nothing on disk in .bss but would bloat .data.

The object file also carries a **symbol table** — names it defines and names it uses but expects someone else to define (undefined symbols) — and **relocations**: patch-lists saying "write the final address of `foo` here once the linker knows it." An object file is a puzzle piece: valid code, unresolved edges.

```bash
g++ -c m.cpp
nm m.o          # T: defined in .text, D: .data, B: .bss, U: undefined
size m.o        # bytes per section — note .bss counted but not stored
```

## fact: The linker is a symbol matchmaker, and the ODR is its blind spot
tags: toolchain, compiler, linker
track: core

The linker merges object files: every *undefined* symbol must find exactly one *defined* symbol. Zero matches → `undefined reference`; two strong definitions → `multiple definition`. But C++'s **One Definition Rule** asks for more than the linker checks: the same inline function or class compiled into different TUs must be *identical*. If two TUs saw different versions of a header, the linker silently keeps one copy — which TUs' code now disagrees with — and you get undefined behavior, not an error.

C++ function names are **mangled** — `checksum(const char*)` becomes `_Z8checksumPKc` — encoding types into the symbol so overloads coexist and link-time type mismatches surface. `extern "C"` switches a function to plain C naming, the lingua franca for calling C libraries or being called from them. **Static linking** copies library code into your binary (self-contained, larger, needs relink to update); **dynamic linking** records a dependency on a `.so`/`.dylib` resolved at load time (shared pages, independently updatable — and a new failure mode at deploy: the missing or wrong-version library).

```cpp
extern "C" int checksum(const char* s);  // link against C symbol "checksum"
// without extern "C": undefined reference — C++ looked for _Z8checksumPKc
```

## fact: LTO optimizes across files, PGO optimizes for reality
tags: toolchain, compiler, lto
track: core

Normal compilation optimizes one TU at a time: a function in `a.cpp` called from `b.cpp` can't be inlined, because the compiler never sees both bodies together. **Link-Time Optimization** (`-flto`) fixes this by storing IR (not just machine code) in object files and re-running optimization over the *whole program* at link time — cross-TU inlining, dead code stripping across files, devirtualization. Cost: much slower links, higher memory. It also quietly weakens the "just put it in a .cpp file" firewall for build times.

**Profile-Guided Optimization** attacks a different blindness: the compiler guesses which branches are hot. PGO replaces guesses with measurements — build instrumented (`-fprofile-generate`), run a representative workload, rebuild with the profile (`-fprofile-use`). The compiler then lays out hot paths contiguously (better i-cache), inlines the calls that actually happen, and moves cold error paths out of the way. Real-world gains of 5–15% are common; the catch is keeping the training workload representative.

```bash
g++ -O2 -flto a.cpp b.cpp -o app          # whole-program view at link
g++ -O2 -fprofile-generate app.cpp && ./app train.dat
g++ -O2 -fprofile-use app.cpp -o app      # optimize for measured behavior
```

## fact: Debuggers and profilers see your binary through side tables
tags: toolchain, compiler, debugging
track: core

The CPU executes raw machine code; everything a debugger shows you — file:line, variable names, types, stack frames — comes from **DWARF** debug info that `-g` embeds alongside the code, mapping instruction addresses back to source. `-g` doesn't slow the generated code down; it just makes the binary bigger (and production builds often `strip` it into a separate file, keeping it server-side to symbolicate crash dumps). `-O2 -g` is legal but surreal to step through: variables are "optimized out", lines execute out of order, inlined calls have no frame.

Profilers like `perf` mostly don't trace — they **sample**: interrupt the process a few thousand times per second and record the instruction pointer plus call stack. Hot code is simply where samples pile up. To get stacks and names they need the symbol table and either frame pointers (`-fno-omit-frame-pointer`, the pragmatic prod default) or DWARF unwind info. A profile full of `[unknown]` means the toolchain stripped the map, not that the CPU is idle.

```bash
g++ -O2 -g -fno-omit-frame-pointer app.cpp -o app
perf record -g ./app     # sample IP + call stacks
perf report              # samples aggregated per function
```
