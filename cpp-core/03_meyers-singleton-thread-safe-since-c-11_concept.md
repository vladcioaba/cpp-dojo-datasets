## fact: Meyers Singleton — thread-safe since C++11
tags: patterns, singleton

A function-local `static` is initialized exactly once, and since C++11 the standard guarantees that initialization is thread-safe ("magic statics"). No locks, no `std::call_once`, no double-checked locking.

```cpp
Config& Config::instance() {
    static Config cfg;   // initialized once, thread-safe
    return cfg;
}
```

It also solves the static initialization order fiasco: the object is created on first use, not at some unspecified point before `main`.
