## exercise: Meyers Singleton
tags: patterns, singleton

Write the body of `instance()` so that `Config` is a lazy, thread-safe singleton. Use a function-local static named `cfg`.

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
