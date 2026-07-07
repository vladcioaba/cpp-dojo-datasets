## exercise: RAII file guard
tags: raii, patterns

Write a class `File` that opens a `FILE*` with `fopen(path, "r")` in its constructor (member named `f`, parameter named `path`, type `const char*`) and closes it with `fclose(f)` in its destructor if it is non-null.

```cpp
class File {
    FILE* f;
public:
    File(const char* path) : f(fopen(path, "r")) {}
    ~File() { if (f) fclose(f); }
};
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    { File f("/etc/hosts"); }
    std::puts("PASS");
}
```
