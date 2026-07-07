## exercise: Meyers Singleton
tags: patterns, singleton

Write the body of `instance()` so that `Config` is a lazy, thread-safe singleton. Use a function-local static named `cfg`.

hint: A function-local `static` is constructed exactly once, on the first call — that is the entire trick.
hint: Return a reference to a local `static Config`; since C++11 its initialization is guaranteed thread-safe.

```cpp
// starter
class Config {
public:
    static Config& instance();
private:
    Config() = default;
};
```

```cpp
Config& Config::instance() {
    static Config cfg;
    return cfg;
}
```

```cpp
// harness
#include <cstdio>
class Config {
public:
    static Config& instance();
private:
    Config() = default;
};
//__USER__
int main() {
    if (&Config::instance() != &Config::instance()) return 1;
    std::puts("PASS");
}
```

**Editorial:** The Meyers singleton relies on a function-local static: it is constructed lazily on the first call, and since C++11 that initialization is guaranteed thread-safe (the compiler inserts a one-time guard). Returning it by reference yields a single shared instance with no manual locking. The drill teaches lazy, thread-safe single-instance initialization.
