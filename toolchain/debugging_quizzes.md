## quiz: Match each fault address to its bug class
tags: debugging, segfault
track: core
difficulty: medium

Three programs die with SIGSEGV at these fault addresses:

```text
A: 0x0000000000000018
B: 0x00007ffc9a3feff8   (just below the thread's stack region)
C: 0x4141414141414141
```

- [ ] A: stack overflow, B: null deref, C: use-after-free
- [x] A: null pointer + member offset, B: stack overflow, C: wild pointer from overwritten data
- [ ] A: wild pointer, B: use-after-free, C: null pointer + member offset
- [ ] A: use-after-free, B: wild pointer, C: stack overflow

> Small fault addresses like 0x18 are a null pointer plus an offset — `p->field` where the field sits 24 bytes into the struct. An address just past the stack region is the guard page: stack overflow, confirmed by a backtrace of thousands of identical frames. And 0x41 is ASCII 'A' — data ("AAAAAAAA") is being used as a pointer, typically because an overflow overwrote one.

## quiz: Which gdb command stops the program when a variable's value changes?
tags: debugging, gdb
track: core
difficulty: easy

You need to find which code is corrupting `balance` — you don't know where the write happens.

- [ ] break balance
- [x] watch balance
- [ ] display balance
- [ ] catch balance

> `watch balance` sets a watchpoint: the program stops the instant the value changes, no matter which line wrote it — the direct answer to "who is modifying this?". `break` needs a location, `display` merely re-prints an expression at each stop, and `catch` hooks events like `throw` or syscalls, not variables. Related: `rwatch` stops on reads, `awatch` on both.

## quiz: gdb prints "<optimized out>" for a local variable. What does it mean?
tags: debugging, optimized-builds
track: core
difficulty: medium

- [ ] The variable's memory was corrupted, so gdb refuses to print it
- [ ] The debug info is broken; recompiling with -g will fix it at any -O level
- [x] At this point the value is not stored in any register or memory slot — the optimizer eliminated it
- [ ] The variable is in another thread's stack frame

> `<optimized out>` is honest debug info: the compiler proved it didn't need to materialize the value here (folded it into another computation, or its live range ended), so there is nothing for gdb to read. It is not corruption and not a gdb bug. The fix is building that file with `-O0` or `-Og` — `-g` alone doesn't help, because `-g` never changes code generation.

## quiz: A crash reproduces every run — until you attach gdb. Why does it vanish?
tags: debugging, heisenbug
track: core
difficulty: medium

- [ ] gdb inserts memory barriers around shared variables, fixing the race
- [ ] Compiling with -g adds synchronization to std::thread
- [x] The bug is timing-sensitive (likely a race), and the debugger's interference reshuffles thread scheduling so the window never opens
- [ ] gdb runs the program single-threaded

> A race is a timing window, often nanoseconds wide. A debugger stopping threads at breakpoints — or even an added printf, with its milliseconds and internal stdio lock — changes the schedule enough that the interleaving never occurs. gdb adds no barriers and doesn't force single-threading; the bug is still there, just hidden. The right tool is TSan (`-fsanitize=thread`), which reports the race without needing it to misfire.

## quiz: An ASan report says heap-use-after-free and shows two stacks besides the bad access. What do they show?
tags: debugging, asan
track: core
difficulty: medium

```text
==4242==ERROR: AddressSanitizer: heap-use-after-free ...
READ of size 4 at 0x6020000000b0 thread T0
    #0 ...
freed by thread T0 here:
    #0 ...
previously allocated by thread T0 here:
    #0 ...
```

- [ ] The two most recent function calls before the crash
- [ ] The same access from two different threads that raced
- [x] Where the memory was freed, and where it was originally allocated
- [ ] The shadow-memory mapping of the address

> The report is the object's biography: the access stack shows the illegal read, "freed by" shows the deallocation that made it illegal, and "previously allocated by" shows where the object was born. Together they settle the two competing theories of any use-after-free — "freed too early" vs "pointer kept too long" — without any logging or guesswork.

## quiz: Your program segfaulted but no core file appeared. What do you check first?
tags: debugging, core-dumps
track: core
difficulty: easy

- [ ] Whether the binary was compiled with -g
- [x] ulimit -c — the core size limit is usually 0 by default
- [ ] Whether gdb is installed
- [ ] Whether ASLR is enabled

> Most systems default the core size limit to 0, so no core is ever written; `ulimit -c unlimited` enables them for the shell. Only then do location questions arise (`/proc/sys/kernel/core_pattern`, or `coredumpctl` on systemd distros). `-g` affects how *readable* the core is in gdb, not whether it is dumped, and ASLR/gdb have nothing to do with core creation.

## quiz: A bug appears at -O2 but vanishes at -O0. What is the most likely cause?
tags: debugging, optimized-builds
track: core
difficulty: medium

- [ ] An optimizer bug in the compiler — file a report
- [ ] -O2 strips debug info, which changes program behavior
- [x] Undefined behavior in your code that the optimizer's assumptions exposed
- [ ] -O2 uses a smaller stack, causing overflow

> Optimizers transform code under the assumption the program has no UB — signed overflow, out-of-bounds reads, strict-aliasing violations. Code whose UB was harmless at -O0 breaks when a pass exploits the assumption (e.g. deleting a "redundant" null check or overflow test). Genuine optimizer bugs exist but are rare; debug info never affects codegen. First move: UBSan and ASan on the release configuration, not a compiler bug report.

## quiz: What must you have before git bisect can find the commit that broke things?
tags: debugging, bisect
track: core
difficulty: easy

- [ ] A branch containing only the suspect commits
- [x] A reliable test that classifies any given commit as good or bad
- [ ] CI history for every commit in the range
- [ ] The name of the file containing the bug

> Bisecting is a binary search over history, and a binary search is only as good as its comparison: you need a check that deterministically answers "does this commit have the bug?" — plus one known-good and one known-bad commit to bound the range. A flaky reproducer poisons the search, since one wrong answer sends the search into the wrong half. With a script, `git bisect run ./repro.sh` automates the whole hunt (exit 0 = good, 1-127 = bad, 125 = skip).

## quiz: You set a watchpoint and the program now runs orders of magnitude slower. Why?
tags: debugging, breakpoints
track: core
difficulty: hard

```text
(gdb) watch cfg.entries
Watchpoint 2: cfg.entries
```

- [ ] Watchpoints always cost this much — that is the price of watching memory
- [ ] The watched variable is in a shared library, forcing gdb to reload symbols
- [x] gdb fell back to a software watchpoint, single-stepping the program and checking the value after each step
- [ ] The watchpoint is triggering repeatedly without stopping

> Note the message says "Watchpoint", not "Hardware watchpoint". Hardware watchpoints use the CPU's debug registers (four on x86) and run at full speed — but when they're exhausted, or the watched expression is too large or complex for a register, gdb silently falls back to software watching: single-step, check, repeat. Fixes: watch a specific scalar address (`watch -l`, or `watch *&var`), reduce the number of watchpoints, and see `show can-use-hw-watchpoints`.

## quiz: A server process is hung and using 0% CPU. Which tool answers "what is it stuck on?" fastest?
tags: debugging, hangs
track: core
difficulty: medium

- [ ] perf record — sample where it spends its CPU time
- [x] strace -p PID (or gdb attach + thread apply all bt) — see the blocked syscall
- [ ] valgrind — rerun the server under memcheck
- [ ] Restart it under -O0 and wait for the hang again

> Zero CPU means the process isn't spinning — it is blocked in a syscall. `strace -p` shows the un-returning call immediately: `read(7,` means waiting on fd 7 (then `lsof -p` tells you what fd 7 is), `futex(` means a lock or condition variable — that one's for `gdb -p` and `thread apply all bt` to expose the deadlock. perf profiles CPU use, which is the right tool for the *opposite* hang (100% CPU spin); valgrind and rebuilds both lose the live, already-hung state.
