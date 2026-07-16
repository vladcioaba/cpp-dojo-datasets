## fact: Sanitizers are runtime bug detectors — one recompile, four families
tags: toolchain, sanitizers
track: core

Sanitizers instrument your code at compile time so entire bug classes crash loudly with a stack trace instead of corrupting memory silently. **ASan** (`-fsanitize=address`): heap/stack buffer overflows, use-after-free, double-free, leaks — roughly 2x slowdown. **UBSan** (`-fsanitize=undefined`): signed overflow, null deref, misaligned access, bad shifts, out-of-range enum loads — so cheap (tens of percent) some shops ship it enabled. **TSan** (`-fsanitize=thread`): data races and lock-order inversions — the only practical race detector, at 5–15x slowdown and heavy memory. **MSan** (`-fsanitize=memory`): reads of uninitialized memory — powerful but demands *every* linked library be instrumented, so it's mostly a dedicated-CI tool.

Rules of thumb: ASan+UBSan combine and belong in every test run; TSan must run alone (it and ASan/MSan are mutually exclusive); a sanitizer only checks code that *executes*, so coverage is only as good as your tests.

```bash
g++ -g -O1 -fsanitize=address,undefined tests.cpp -o tests   # the CI staple
g++ -g -O1 -fsanitize=thread mt_tests.cpp -o mt_tests        # races, alone
./tests   # bug -> immediate report with stack trace, nonzero exit
```

## fact: Static analysis finds bugs without running anything — if you let it fail the build
tags: toolchain, static-analysis
track: core

Static analysis inspects source instead of executing it. Tier one is the compiler itself: `-Wall -Wextra` is the floor (despite the name, `-Wall` is far from all), and the culture that matters is **`-Werror` — warnings fail the build**. A warning that doesn't fail CI is a warning everyone learns to scroll past; at zero-warning steady state, every new one is signal. Adopting `-Werror` late is painful, so ratchet: fix a category, then lock it.

Tier two is **clang-tidy**: hundreds of checks that know C++ idioms, not just syntax — use-after-move, dangling `string_view`, missing `override`, modernize-* and performance-* families — many with automatic fixes (`--fix`). It runs on one TU at a time using `compile_commands.json` (the CMake lesson's export flag is the setup). Beyond it sit whole-program analyzers (clang static analyzer, Coverity). Static analysis and sanitizers are complements, not rivals: tidy sees paths your tests never execute; sanitizers see truths only visible at runtime.

```bash
# .clang-tidy at repo root; version it like code
clang-tidy --checks='bugprone-*,performance-*,modernize-use-override' \
           -p build/ src/engine.cpp
g++ -Wall -Wextra -Werror -c src/engine.cpp   # warnings are errors
```

## fact: vcpkg and conan bring package management to a language without one
tags: toolchain, package-managers
track: core

C++ has no built-in equivalent of pip — the void is filled by system packages (versions frozen by your distro), vendoring/FetchContent (you compile everything), and two real package managers. **vcpkg** (Microsoft, curated central registry): in manifest mode you commit a `vcpkg.json` naming dependencies; a CMake toolchain file makes `find_package` find them, building from source per **triplet** (`x64-linux`, `arm64-osx`) with binary caching. **conan** (JFrog, decentralized): recipes in `conanfile.txt/.py`, **profiles** capturing compiler/stdlib/flags, and first-class *prebuilt binary* packages — it downloads a compatible binary if one matches your profile, else builds.

Why this is hard enough to need tooling: a C++ binary package is only compatible if compiler, standard library, CXX standard, build type, and ABI-relevant flags all line up — that's exactly what triplets and profiles encode. Rough heuristic: vcpkg for CMake-centric simplicity, conan for binary distribution and org-internal registries. Either beats hand-rolled `ExternalProject` scripts.

```bash
# vcpkg manifest mode: vcpkg.json next to CMakeLists.txt
# { "dependencies": [ "fmt", "boost-asio" ] }
cmake -S . -B build \
  -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
# deps resolve during configure; find_package(fmt) now works
```

## fact: Build speed is a tooling problem before it's a hardware problem
tags: toolchain, build-speed
track: core

Three multipliers, in order of adoption. **ninja** over make: same work, better engine — accurate dependency handling, maximal parallelism, and near-instant no-op builds (make can spend seconds just stat-ing files; ninja was built for the rebuild-after-one-edit loop). With CMake it's one flag: `-G Ninja`. **ccache** caches compilations: it hashes the preprocessed input and flags, and on a hit returns the previous object file instead of invoking the compiler — transformative when switching branches, rebuilding after clean, or across CI runs with a shared cache (`CMAKE_CXX_COMPILER_LAUNCHER=ccache`).

**Unity builds** attack C++'s structural cost: every TU re-parses every header it includes — the same megabytes of `<vector>` and `<string>` parsed hundreds of times. Concatenating groups of .cpp files (`CMAKE_UNITY_BUILD=ON`) amortizes that, often 2–5x on full builds. Costs: `static` globals and anonymous namespaces from different files now share a TU and can collide, missing-include bugs hide (a header leaks in from a sibling), and touching one file recompiles its whole group. The deeper fix for header cost is precompiled headers or C++20 modules.

```bash
cmake -S . -B build -G Ninja -DCMAKE_CXX_COMPILER_LAUNCHER=ccache
cmake --build build          # cold: compiler speed; warm: cache speed
ccache -s                    # hit/miss statistics
```

## fact: API compatibility is about recompiling; ABI is about not recompiling
tags: toolchain, abi
track: core

**API** is the source-level contract — if it changes, your code fails to *compile*. **ABI** (Application Binary Interface) is the machine-level contract between *already-compiled* binaries: struct layouts, vtable order, calling conventions, name mangling, inline function semantics. ABI breaks don't fail cleanly — the call site computes field offsets from the headers *it* was built with, so a mismatch reads garbage at runtime. Change a class's members, reorder virtuals, or flip a size-affecting `#define`, and every binary compiled against the old layout is silently wrong.

Passing `std::string` across a `.so` boundary is the classic hazard: caller and library must agree on the *exact* internal layout, which varies by standard library (libstdc++ vs libc++), by vendor version (libstdc++'s C++11 dual-ABI split — `_GLIBCXX_USE_CXX11_ABI`), and by build flags (debug/iterator-checked modes). The escape hatch is the **C ABI** — the one contract that's stable per platform: `extern "C"` functions, primitive types, and opaque pointers. It's why plugin systems and cross-language FFI boundaries are C even when both sides are C++.

```cpp
// fragile across .so boundary:
std::string get_name();                     // layout must match exactly
// stable C ABI surface:
extern "C" engine_t* engine_create(void);   // opaque handle
extern "C" int engine_name(engine_t*, char* buf, size_t len);
```

## fact: Headers work because of inline — and break because of ODR
tags: toolchain, odr
track: core

A function *defined* in a header gets compiled into every TU that includes it — normally a `multiple definition` link error. `inline` (and its implicit forms: member functions defined in-class, templates, `constexpr` functions, C++17 `inline` variables) changes the deal: multiple definitions are allowed, each TU emits a weak/COMDAT copy, and the linker keeps one. The One Definition Rule's fine print is the contract: all those definitions must be *token-for-token identical*. If two TUs saw different versions — a stale object file, an `#ifdef` that differs, two versions of a vendored header — the linker still silently picks one, and the TUs built against the other are now undefined behavior. Symptom: works in Debug, crashes in Release, changes with link order.

Related hygiene at `.so` boundaries: ELF exports every symbol by default, bloating symbol tables, slowing loads, and letting duplicate symbols from different libraries interpose each other. Shared libraries should build with `-fvisibility=hidden` and export an explicit annotated API — which also keeps two libraries' private copies of the same inline function from colliding.

```cpp
// header — legal in many TUs, but every TU must see identical tokens
inline int price_ticks(double px) {
#ifdef FAST_MATH                    // differs per TU? silent ODR violation
    return int(px * 100);
#else
    return std::lround(px * 100);
#endif
}
```

## fact: Cross-compilation splits the world into host and target
tags: toolchain, cross-compilation
track: core

Cross-compiling means the **host** (machine running the compiler, say x86-64 Linux) differs from the **target** (machine running the binary — an ARM board, an Android phone). You need three things. A **cross toolchain**: compiler, assembler, linker emitting target code — named by **triplet** (`aarch64-linux-gnu-g++`), or clang with `--target=`. A **sysroot**: a directory mirroring the target's `/usr/include` and `/usr/lib`, because your program must compile against the *target's* libc and libraries, not the host's — most "undefined symbol" and "wrong ELF class" errors are sysroot problems. And a build system told about the split: in CMake, a **toolchain file** passed as `-DCMAKE_TOOLCHAIN_FILE=` sets `CMAKE_SYSTEM_NAME`, the compilers, sysroot, and find-root rules so `find_package` searches the sysroot instead of the host.

The subtle traps: anything *executed during the build* (code generators, `try_run` probes) must be built for the host, not the target; and pkg-config/find modules silently leaking host paths into a target build.

```cmake
# aarch64.toolchain.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)
set(CMAKE_C_COMPILER   aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)
set(CMAKE_SYSROOT /opt/sysroots/aarch64-target)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)   # search sysroot, not host
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```

## fact: Reproducible builds: same source in, bit-identical binary out
tags: toolchain, reproducible-builds
track: core

A build is **reproducible** when the same source and toolchain produce a *bit-identical* binary — anywhere, anytime, by anyone. The classic saboteurs: `__DATE__`/`__TIME__` macros, absolute build paths baked into debug info (`-ffile-prefix-map=/home/ci=.` fixes that), archive timestamps, and nondeterministic link or codegen ordering. The `SOURCE_DATE_EPOCH` convention pins every embedded timestamp. The prerequisite is **hermeticity**: the build depends only on *declared* inputs — pinned compiler version, pinned dependencies, no "whatever is in /usr/include today". Docker images with pinned toolchains get most of the way; Bazel-style systems enforce it by sandboxing every action, which is also what makes remote caching sound — a cache key is trustworthy only if it captures *all* inputs.

Why regulated and HFT shops insist on it: you must be able to prove the binary trading in production corresponds exactly to an audited source revision, reproduce last Tuesday's build to debug a live incident, and detect supply-chain tampering — a rebuilt-from-source binary that doesn't match the deployed hash is either a hermeticity bug or a very bad day.

```bash
export SOURCE_DATE_EPOCH=1700000000
g++ -O2 -g -ffile-prefix-map=$PWD=. -c engine.cpp
# two clean builds, then verify:
sha256sum build-a/engine.o build-b/engine.o   # hashes must match
```
