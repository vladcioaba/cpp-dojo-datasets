## fact: Debugging is a protocol — reproduce, minimize, bisect, change one thing
tags: debugging, mindset
track: core

Debugging feels like intuition but works like science, and the protocol has four steps. **Reproduce** first: a bug you cannot trigger on demand cannot be proven fixed. Pin down everything — exact input, environment variables, thread count, random seed. A "sometimes" bug becomes an "always" bug once you find the variable you weren't controlling. **Minimize** second: cut the input or the code in half, keep whichever half still fails, repeat. A 40-line reproducer is worth hours of staring at 40,000 lines, and tools like `creduce` automate this for compiler-ish bugs. **Bisect** third when the question is *"which change broke it?"*: `git bisect` binary-searches history, so 4,000 commits need only ~12 tests, and `git bisect run` makes it fully automatic given a script that exits 0 for good and nonzero for bad. Finally, **change one thing at a time**: every experiment should test a single hypothesis, and if you changed two things and the crash moved, you learned nothing. Keep notes — the fifth hypothesis has a way of quietly contradicting the second.

```text
$ git bisect start
$ git bisect bad                  # HEAD is broken
$ git bisect good v2.1            # last release that worked
Bisecting: 512 revisions left to test after this (roughly 9 steps)
$ git bisect run ./repro.sh       # exit 0 = good, 1-127 = bad, 125 = skip commit
...
abc1234 is the first bad commit
$ git bisect reset
```

The rest of this track is tooling. Tools make the protocol fast; they never replace it.

## fact: gdb/lldb essentials — ten commands cover ninety percent of sessions
tags: debugging, gdb
track: core

Build with `-g` so the compiler emits DWARF debug info mapping machine code back to source lines and variables — `-g` by itself does not slow your program down, it just makes the binary bigger. What ruins debugging is optimization, so debug builds use `-O0` (nothing optimized, every variable inspectable) or `-Og` (light optimization tuned to stay debuggable — GCC's recommended edit-compile-debug default).

The core vocabulary, identical in spirit across gdb and lldb:

```text
$ g++ -g -O0 parse.cpp -o parse
$ gdb ./parse
(gdb) break parse.cpp:42        # b — stop when this line is reached
(gdb) run data.txt              # r — start the program with arguments
(gdb) next                      # n — one line, stepping OVER calls
(gdb) step                      # s — one line, stepping INTO calls
(gdb) print row.size()          # p — evaluate any expression, even calls
(gdb) bt                        # backtrace: the call chain that got you here
(gdb) frame 2                   # jump to frame #2 from bt
(gdb) info locals               # all locals of the current frame
(gdb) finish                    # run until the current function returns
(gdb) continue                  # c — resume until the next stop
```

Two habits complete the toolkit: `print` evaluates real expressions — `p rows[i].name`, even method calls — so you rarely need extra logging once stopped; and `bt` then `frame N` is the standard crash autopsy: find the deepest frame that is *your* code and inspect its locals there. For a process that is already running (a hung server, say), `gdb -p PID` attaches without restarting it.

lldb, the default on macOS, uses the same words — `run`, `next`, `step`, `continue`, `bt`, `print` — with `frame select 2` for frame hopping, and `b parse.cpp:42` works as a breakpoint shorthand. Learn these ten and any C++ codebase on any platform is steppable; everything else in gdb is refinement of *where to stop*, which is the next lesson.

## fact: Breakpoints beyond 'break' — conditions, watchpoints, catchpoints
tags: debugging, breakpoints
track: core

Plain breakpoints stop too often. The refinements are what make a debugger surgical.

**Conditional breakpoints** stop only when an expression is true: `break process.cpp:88 if id == 42` skips the 10,000 boring iterations. Retrofits work too: `condition 3 attempts > 5` adds a condition to existing breakpoint 3, and `ignore 3 100` skips its next 100 hits — perfect for "it crashes on roughly the hundredth call". `tbreak` is a one-shot breakpoint that deletes itself after the first hit (gdb's `start` command is exactly `tbreak main` + `run`).

**Watchpoints** flip the question from *where* to *what*: `watch queue.size_` stops the program the instant that value changes, no matter which line changed it — the definitive answer to "who is corrupting this variable?" On x86 these use the CPU's four hardware debug registers, so they run at full speed; when gdb can't use them (too many, or the watched expression is too complex) it falls back to software watchpoints that single-step the whole program, orders of magnitude slower. `rwatch` stops on reads, `awatch` on both.

**Catchpoints** hook events instead of locations: `catch throw` stops at the exact throw site of any exception (before the stack unwinds away the evidence), `catch catch` at the handler, and `catch syscall openat` at the OS boundary.

```text
(gdb) break process.cpp:88 if id == 42
(gdb) watch queue.size_
Hardware watchpoint 2: queue.size_
(gdb) catch throw
Catchpoint 3 (throw)
```

## fact: Reading a backtrace — inlined frames, ?? frames, and mangled names
tags: debugging, backtrace
track: core

A backtrace is the call chain, newest frame first — and frame #0 is where the program *stopped*, which is often not where the *bug* is. Cultivate the habit of scanning downward for the first frame in *your* code.

```text
(gdb) bt
#0  Table::at (this=0x0, i=7) at table.h:31
#1  0x000055e4a1c22b10 in Report::total () at report.cpp:58
#2  run () at main.cpp:19
#3  0x00007f21e4a2b083 in ?? ()
```

Three things trip people up. **Inlined frames**: in optimized builds a call like `Report::total` may not exist as a real function anymore, but modern DWARF records the inlining, so gdb synthesizes a frame for it — you may see several frames sharing one PC address, all legitimate. **`?? ()` frames** mean gdb has an address but no symbol for it: a stripped binary, a library without debug info (install the distro's debuginfo package, or point gdb at unstripped builds), or JIT-generated code. A backtrace that is *all* `??` with nonsense addresses usually means the stack itself is corrupted — see the last lesson. **Mangled names**: gdb demangles automatically, but raw symbols like `_ZN5Table2atEm` still leak out of `nm`, linker errors, and `backtrace_symbols()` logs. Pipe them through `c++filt`:

```text
$ echo _ZN5Table2atEm | c++filt
Table::at(unsigned long)
```

For multithreaded programs, `thread apply all bt` dumps every thread's stack — the first command to run on any deadlock.

## fact: Core dumps — post-mortem debugging, and why reruns lie
tags: debugging, core-dumps
track: core

A core dump is a snapshot of a dead process — its full memory and registers at the instant of the fatal signal. It turns "it crashed last night" into a debuggable artifact.

They are usually disabled: `ulimit -c` prints the maximum core size and on most systems it defaults to `0`, meaning nothing is ever written. That is the first thing to check when a core is missing. Where cores land is decided by `/proc/sys/kernel/core_pattern` — a filename pattern, or a pipe into a collector: systemd distros route cores to `systemd-coredump`, where `coredumpctl list` shows recent crashes and `coredumpctl gdb` opens the latest one directly.

```text
$ ulimit -c unlimited            # allow cores in this shell
$ ./server                       # ... Segmentation fault (core dumped)
$ gdb ./server core              # post-mortem session
(gdb) bt                         # the frozen moment of death
(gdb) frame 2
(gdb) print *request             # inspection works: memory is all there
(gdb) continue
The program is not being run.    # but execution is over — inspect only
```

Post-mortem debugging matters because of a cruel property of memory bugs: *"it works when I rerun it."* ASLR shuffles addresses, heap layout shifts, thread timing differs — so the same use-after-free reads harmless garbage on the rerun and crashes only under load at 3 a.m. The core preserves the one execution you could never reproduce; treat it as evidence, not an anecdote.

## fact: Segfault anatomy — the fault address names the bug class
tags: debugging, segfault
track: core

SIGSEGV reports *which address* was touched illegally, and that number is a diagnosis. Learn the patterns:

```text
fault address              prime suspect
-------------              -------------
0x0                        null pointer dereferenced directly
0x10, 0x18, small values   null pointer + offset: p->field or p[i] via p == nullptr
just below the stack       stack overflow: bt shows thousands of identical frames
0x4141414141414141         wild pointer: data ("AAAAAAAA"!) interpreted as a pointer
plausible heap address     use-after-free where the allocator returned pages to
                           the OS — or a WRITE to a read-only page (string literals)
pc == fault address        execution jumped through a corrupt function pointer
```

The small-offset rule is exact. Given `struct Session { long id; long flags; long counter; };` and `Session* s = nullptr`, reading `s->counter` computes `nullptr + 16` — the fault address is `0x10`, the member's offset. The address tells you not just "null pointer" but *which member access* died.

Stack overflows are equally recognizable: the fault lands in the guard page just beyond the stack region (in one measured run, a stack local lived at `0x16fa82c9c` and the fault hit `0x16f287ff8` — almost exactly the 8 MB stack size below it), and the backtrace is one function repeated until gdb gives up.

Poison patterns are gifts from debug allocators — values like `0xdeadbeef` or MSVC's `0xdddddddd` mean "freed memory", turning a mystery crash into a use-after-free confession. Production allocators rarely poison, which is one more reason release crashes look stranger than debug ones.

## fact: Debugging optimized builds — <optimized out> and the release-only bug
tags: debugging, optimized-builds
track: core

Attach a debugger to an `-O2` binary built with `-g` and it half-works: stepping jumps forward and backward because the compiler reordered and merged lines; `print x` answers `<optimized out>` because `x` lives in no register or memory slot at this point — the optimizer proved it didn't need to exist; whole functions are missing because they were inlined. None of this is corruption. It is honest debug info about dishonest-looking code. Mitigations: build daily-debug configurations with `-Og`; add `-fno-omit-frame-pointer` so stack walks stay reliable (profilers and crash reporters need it too); remember `-g` is compatible with every `-O` level — it never changes the generated code, only your visibility into it.

The deeper trap is the bug that *only exists* at `-O2`. It is almost never a compiler bug. Optimizers transform code under the assumption that your program has no undefined behavior — so existing UB, harmless-looking at `-O0`, becomes load-bearing:

```cpp
int steps(int start) {
    int n = 0;
    for (int i = start; i > 0; i += i)   // doubling overflows: signed UB
        ++n;
    return n;
}
```

Compiled and run with one mainstream compiler, `steps(1)` prints **31** at `-O0` (the int wrapped negative and the loop exited) and **0** at `-O2` (the optimizer assumed `i > 0` can't be falsified without UB and rewrote the loop). Same source, two answers, zero compiler bugs. So when a bug appears only in release, reach for UBSan and ASan on the *release* configuration before blaming the compiler.

## fact: Reading an ASan report — an object's whole life story
tags: debugging, asan
track: core

You already build tests with `-fsanitize=address`; the skill is reading what it prints. An ASan report for a heap bug is a biography in three stack traces. Here is a real (condensed) report for reading freshly-deleted memory:

```text
==4242==ERROR: AddressSanitizer: heap-use-after-free on address 0x6020000000b0
READ of size 4 at 0x6020000000b0 thread T0
    #0 in main uaf.cpp:5                      <- the illegal touch
0x6020000000b0 is located 0 bytes inside of 4-byte region
freed by thread T0 here:
    #1 in main uaf.cpp:4                      <- the death
previously allocated by thread T0 here:
    #1 in main uaf.cpp:3                      <- the birth
SUMMARY: AddressSanitizer: heap-use-after-free uaf.cpp:5 in main
=>0x602000000080: fa fa 00 00 fa fa[fd]fa fa fa
```

Read it in story order: the object was **allocated** (uaf.cpp:3), then **freed** (uaf.cpp:4), then **read** (uaf.cpp:5). The three stacks localize both halves of every use-after-free argument — "who freed it early?" versus "who kept a stale pointer?" — which is precisely the debate you'd otherwise settle with hours of logging. The "0 bytes inside of 4-byte region" line tells you the access hit the object itself, not a neighboring one (overflow reports say "0 bytes to the right of").

The shadow-byte map at the bottom encodes one byte per 8 application bytes: `00` addressable, `fa` heap redzone, `fd` freed memory — and the bracketed `[fd]` marks your access sitting in freed memory. You rarely need the map; the three stacks are the report.

## fact: Debugging data races — heisenbugs, TSan reports, sleep injection
tags: debugging, tsan
track: core

Races are the canonical *heisenbug*: attach gdb and the crash vanishes. No mystery — a race is a timing window often nanoseconds wide, and a debugger stopping all threads at breakpoints (or even a `printf`, which adds milliseconds and internal stdio locking) reshuffles the schedule so the window never opens. Observation changes timing; timing *is* the bug.

So don't chase races with a stepper — build with `-fsanitize=thread` and run the tests. TSan doesn't need the crash to happen: it tracks the happens-before relation and reports two accesses that *could* have collided, with a stack for each side:

```text
WARNING: ThreadSanitizer: data race (pid=12621)
  Write of size 4 at 0x000102ef8000 by thread T1:
    #0 ... race.cpp:5                       <- the lambda's ++counter
  Previous write of size 4 at 0x000102ef8000 by main thread:
    #0 main race.cpp:6                      <- main's ++counter
  Location is global 'counter' at 0x000102ef8000
  Thread T1 (running) created by main thread at:
    #1 main race.cpp:5
```

Read it as: *this* access raced with *that previous* access, on *this named variable*, and here's where the thread was even created. Both sides in one report — the two stacks are the whole diagnosis.

When you suspect a specific interleaving, use **sleep injection**: drop a `std::this_thread::sleep_for(10ms)` into the suspected window. If the failure becomes reliable, you've proven the interleaving. A sleep that "fixes" it is the same proof — and never, ever the fix.

## fact: Valgrind vs sanitizers — the tool for binaries you cannot rebuild
tags: debugging, valgrind
track: core

Sanitizers instrument at compile time, which is exactly their limit: no source, no sanitizer. Valgrind's **memcheck** instruments the *binary* at runtime — your unmodified executable, the vendor's `.so`, the tool you only ship. That is the deciding question between the two families: *can you recompile?* Yes → ASan at ~2x slowdown. No → memcheck at 10–50x (it also serializes threads, so timing bugs hide under it).

```text
$ valgrind --leak-check=full ./thirdparty_tool
==7723== Invalid read of size 4
==7723==    at 0x4008A1: main (in ./thirdparty_tool)
==7723==  Address 0x5a1c044 is 0 bytes inside a block of size 4 free'd
==7723== LEAK SUMMARY:
==7723==    definitely lost: 96 bytes in 2 blocks
==7723==    indirectly lost: 480 bytes in 12 blocks
==7723==    still reachable: 72,704 bytes in 1 blocks
```

Read leaks in that order: *definitely lost* = no pointer to it anywhere, a true leak; *indirectly lost* = reachable only through leaked blocks (fix the parent); *still reachable* = pointed-to at exit, usually benign singletons. Memcheck also flags uninitialized reads — add `--track-origins=yes` to see where the garbage was born. Blind spot: it puts no redzones on the stack, so overflows *within* a stack frame sail past it (ASan catches those).

The neglected sibling is **massif** (`valgrind --tool=massif`, then `ms_print massif.out.<pid>`): a heap profiler that snapshots usage over time and attributes it to allocation sites — the tool for "no leak reported, but RSS climbs forever", which is usually a cache that only grows.

## fact: strace and /proc — debugging at the syscall boundary
tags: debugging, strace
track: core

Some bugs live below your code: the program hangs, a config file "doesn't exist" though you're staring at it, a socket loops on EAGAIN. For those, watch the syscall boundary — on Linux, `strace` prints every syscall with arguments and return values, no recompile, and `-p` attaches to an already-running process.

The classic: a hung server using **no CPU**. Zero CPU means it's not spinning — it's *blocked in a syscall*, and strace shows which one, forever un-returning:

```text
$ strace -f -p 8123
strace: Process 8123 attached
read(7,                                  <- blocked reading fd 7... but what IS fd 7?
$ lsof -p 8123                           # map fd numbers to real things
server 8123 vlad 7u FIFO ... /run/app/events.pipe    <- a pipe nobody writes to
```

A `futex(...)` line instead means it's waiting on a lock or condition variable — switch to `gdb -p` and `thread apply all bt` for the deadlock. For "file not found", `strace -e trace=file ./tool` prints every path the process *actually* opens — ending arguments about search paths in one line: `openat(AT_FDCWD, "/etc/tool/conf.d/main.cfg", O_RDONLY) = -1 ENOENT`.

`ltrace` does the same at the library-call level. `/proc/PID/` rounds out the kit: `status` shows the state letter (`R` running, `S` sleeping, `D` uninterruptible kernel I/O — the unkillable one, often NFS, `Z` zombie), `fd/` lists descriptors, `wchan` names the kernel wait. macOS equivalents: `dtruss`, `fs_usage`, and `sample PID` for a no-debugger backtrace.

## fact: Stack corruption — why the crash site is not the bug site
tags: debugging, stack-corruption
track: core

Locals, saved registers, and the function's return address share the stack. Overflow a local buffer and you overwrite your neighbors — and the symptom appears wherever the *victim* is used, not where the overflow happened. An off-by-one into an adjacent local makes a variable change value "with no write in sight" (a watchpoint from lesson 3 catches the culprit red-handed). Overwrite the return address and the function's body completes in perfect health, then *returns into garbage* — the crash lands in an unrelated function or in a backtrace full of `??`, and nothing in the visible state points at the guilty buffer. This time-and-place gap between cause and symptom is the defining cruelty of memory corruption.

Stack canaries narrow the gap: `-fstack-protector` (and the stronger `-strong`/`-all` variants) places a secret value between the locals and the return address and verifies it in the function epilogue. This program, built with `-fstack-protector-all`:

```cpp
void fill(int n) {
    char buf[8];
    for (int i = 0; i < n; ++i) buf[i] = 'A';       // n = 24: 16 bytes too far
    std::fprintf(stderr, "body finished normally\n"); // this DOES print
}
int main() { fill(24); }
```

prints its message, then aborts at the return — glibc announces `*** stack smashing detected ***`. Note what that demonstrates: even the canary detects at *epilogue*, not at the overflowing write. Only ASan's stack redzones stop the write itself. The general law of weird bugs: prefer any tool that moves detection closer to the moment of corruption.
