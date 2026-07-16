## fact: CMake doesn't build your code — it generates the thing that does
tags: toolchain, cmake, basics
track: core

CMake is a **build system generator**. You describe your project once in `CMakeLists.txt`; CMake **configures** (reads the description, finds compilers and dependencies, probes the platform) and **generates** native build files — Ninja files, Makefiles, a Visual Studio solution, an Xcode project. The actual compiling is done by that generated build system. This is why CMake won: one description, every platform and IDE.

The three-step flow is worth internalizing because errors live in different steps: a typo'd variable fails at *configure*, a missing source file at *generate/build* start, a compiler error at *build*. `-S` names the source dir, `-B` the build dir; `cmake --build` invokes the underlying tool so you don't have to remember whether it's ninja or make.

```cmake
# CMakeLists.txt — the description
cmake_minimum_required(VERSION 3.20)
project(dojo CXX)
add_executable(app main.cpp)
```
```bash
cmake -S . -B build -G Ninja   # configure + generate
cmake --build build            # delegate to ninja
```

## fact: In modern CMake, the target is the unit — not the variable
tags: toolchain, cmake, targets
track: core

Everything in modern CMake hangs off **targets**: `add_executable()` creates a program, `add_library()` a library (`STATIC` archive, `SHARED` .so/.dll, or `INTERFACE` for header-only — a target with no compiled output at all). Each target carries its own properties: sources, include paths, compile options, definitions, linked dependencies. You then configure targets with the `target_*` command family.

The legacy style — mutating global state like `include_directories()` and `CMAKE_CXX_FLAGS`, which leak into every target defined afterwards — is the main source of unmaintainable CMake. The modern rule: if a command doesn't start with `target_` (or set a property on a specific target), be suspicious. Targets compose like objects; globals compose like goto.

```cmake
add_library(core STATIC engine.cpp orders.cpp)
add_library(mathlib INTERFACE)              # header-only
add_executable(app main.cpp)

target_compile_features(core PUBLIC cxx_std_20)  # per-target, not global
```

## fact: PRIVATE, PUBLIC, INTERFACE — the one mental model CMake runs on
tags: toolchain, cmake, propagation
track: core

`target_link_libraries(A <scope> B)` does two jobs: it links B into A, and it decides what A's *own consumers* inherit. The scopes answer one question — **who needs B?** `PRIVATE`: only A's implementation uses B (B is in A's .cpp files, invisible in A's headers) — consumers of A get nothing. `PUBLIC`: B appears in A's public headers, so anyone compiling against A also needs B's headers and link line — consumers inherit it. `INTERFACE`: only consumers need it, not A itself (rare for linking; the norm for header-only libs).

Getting this right is what makes a dependency graph self-assembling: link the top-level target and every transitive requirement — include dirs, definitions, link libraries — flows in automatically. Getting it wrong shows up as the classic bug: `app` fails to find `json.hpp` because `core` linked the JSON lib `PRIVATE` while exposing it in `core.h`. Rule of thumb: mentioned in your headers → `PUBLIC`; implementation detail → `PRIVATE`.

```cmake
target_link_libraries(core
    PUBLIC  fmt::fmt         # fmt types appear in core's headers
    PRIVATE OpenSSL::Crypto) # used only inside core's .cpp files

target_link_libraries(app PRIVATE core)
# app automatically gets fmt's includes + link line; never sees OpenSSL
```

## fact: Usage requirements — a target ships its own build instructions
tags: toolchain, cmake, usage-requirements
track: core

`target_include_directories`, `target_compile_definitions`, and `target_compile_options` take the same PRIVATE/PUBLIC/INTERFACE scopes, and together they form a target's **usage requirements**: everything a consumer must do to compile against it. A well-written library target is self-describing — consumers just link it and inherit the right `-I`, `-D`, and flags. Nobody downstream hand-maintains include paths.

The canonical pattern: a library's public headers live under `include/`, its internals under `src/`. Consumers need `include/` (→ `PUBLIC`); only the library's own .cpp files see `src/` (→ `PRIVATE`). For header-only libraries everything is `INTERFACE` — the target compiles nothing but still carries requirements.

```cmake
add_library(core STATIC src/engine.cpp)
target_include_directories(core
    PUBLIC  ${CMAKE_CURRENT_SOURCE_DIR}/include   # consumers get -Iinclude
    PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src)      # internals stay internal
target_compile_definitions(core PUBLIC CORE_VERSION=2)

add_library(mathlib INTERFACE)
target_include_directories(mathlib INTERFACE include/)
```

## fact: find_package hands you imported targets — use those, not variables
tags: toolchain, cmake, find-package
track: core

`find_package(fmt REQUIRED)` locates a dependency that's already installed on the system. It works in two modes: **Config mode** — the package installed its own `fmtConfig.cmake` describing itself (the modern norm); **Module mode** — CMake or your project supplies a `FindXxx.cmake` that goes hunting (how legacy libs like `FindZLIB` work). `REQUIRED` makes configure fail fast instead of exploding later at link.

What a good find gives you is an **imported target** like `fmt::fmt` — a first-class target carrying the library's full usage requirements: headers, link line, transitive deps, per-config Debug/Release variants. Link it and everything propagates, exactly like your own targets. The old style — pasting `${FMT_INCLUDE_DIRS}` and `${FMT_LIBRARIES}` variables around — loses all of that and is the second great source of broken CMake. The `::` in a name is your quality signal: it's a target, and a typo becomes a configure-time error instead of a weird link failure.

```cmake
find_package(fmt REQUIRED)              # Config mode: finds fmtConfig.cmake
find_package(Threads REQUIRED)          # Module mode: FindThreads.cmake

target_link_libraries(app PRIVATE fmt::fmt Threads::Threads)
# imported targets propagate includes/flags; no *_INCLUDE_DIRS variables
```

## fact: FetchContent builds your dependencies from source, inside your build
tags: toolchain, cmake, fetchcontent
track: core

`find_package` assumes the dependency is already installed — someone provisioned the machine. **FetchContent** removes that assumption: at configure time it downloads the dependency's source (typically a git tag) and folds it into your build via `add_subdirectory`, so its targets — the same `fmt::fmt` — become available as if you'd written them. Clean clone → `cmake -S . -B build` → everything just builds. No system state, exact pinned versions, dependencies compiled with *your* flags and sanitizers.

Tradeoffs: you compile dependencies yourself (first build is slower — GoogleTest is fine, Qt is not), and the dep's CMake must be well-behaved as a subproject. Common hybrid: `find_package` for heavyweight system libs, FetchContent for small pinned ones — or `FetchContent_Declare(... FIND_PACKAGE_ARGS)` to try the installed copy first and fall back to source. Always pin an exact tag or commit hash: floating branches are how builds stop being reproducible.

```cmake
include(FetchContent)
FetchContent_Declare(fmt
    GIT_REPOSITORY https://github.com/fmtlib/fmt.git
    GIT_TAG        10.2.1)               # pin exactly — never a branch
FetchContent_MakeAvailable(fmt)

target_link_libraries(app PRIVATE fmt::fmt)   # same target as find_package
```

## fact: Generator expressions answer questions CMake can't answer yet
tags: toolchain, cmake, generator-expressions
track: core

`if()` runs at configure time — but some facts aren't settled then. With multi-config generators (Visual Studio, Xcode), one configure serves Debug *and* Release; the config is chosen at build time. **Generator expressions** — `$<...>` — are placeholders evaluated during the generate step, per configuration: `$<CONFIG:Debug>` is `1` in Debug builds and `0` otherwise, and the conditional form `$<condition:payload>` expands the payload only when the condition holds.

This is the correct tool for per-config flags and definitions — `if(CMAKE_BUILD_TYPE STREQUAL "Debug")` is the bug, because `CMAKE_BUILD_TYPE` is empty or meaningless under multi-config generators. Genexes also handle per-compiler flags (`$<CXX_COMPILER_ID:MSVC>`) and the build-vs-install include-path split. They compose like nested function calls and turn unreadable fast — keep them short, or bind them to a name with a variable.

```cmake
target_compile_definitions(core PRIVATE
    $<$<CONFIG:Debug>:DOJO_TRACE_LOGGING>)     # Debug builds only

target_compile_options(core PRIVATE
    $<$<CXX_COMPILER_ID:MSVC>:/W4>
    $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra>)
```

## fact: Out-of-source builds, CMAKE_BUILD_TYPE, and the two files tooling needs
tags: toolchain, cmake, workflow
track: core

Always build **out of source**: all generated state goes in `build/`, your tree stays clean, and you can keep several configurations side by side (`build-debug/`, `build-asan/`) — deleting one is a true clean build. For single-config generators, `CMAKE_BUILD_TYPE` selects the flag set: `Debug` (`-g`, no optimization), `Release` (`-O3 -DNDEBUG`), `RelWithDebInfo` (`-O2 -g -DNDEBUG` — the production-debugging sweet spot). Trap: unset means *no optimization flags at all*; benchmarking such a build is meaningless. Set it explicitly.

Two JSON files tame the workflow. `CMAKE_EXPORT_COMPILE_COMMANDS=ON` writes **compile_commands.json** — the exact compile command per file — which is how clangd, clang-tidy, and IDEs understand your code; symlink it to the repo root. **CMakePresets.json** (committed) names your configurations so "the asan build" is one flag for every developer and CI, instead of a wiki page of incantations.

```cmake
# CMakePresets.json (excerpt)
{ "name": "release", "binaryDir": "build-release",
  "generator": "Ninja",
  "cacheVariables": { "CMAKE_BUILD_TYPE": "Release",
                      "CMAKE_EXPORT_COMPILE_COMMANDS": "ON" } }
```
```bash
cmake --preset release && cmake --build build-release
```
