## exercise: RAII file guard
tags: raii, patterns

Write a class `File` that opens a `FILE*` with `fopen(path, "r")` in its constructor (member named `f`, parameter named `path`, type `const char*`) and closes it with `fclose(f)` in its destructor if it is non-null.

hint: Tie the resource's lifetime to the object's lifetime — acquire in the constructor, release in the destructor.
hint: Open with `fopen` in the constructor's initializer list; guard the `fclose` with a null check in the destructor.

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

**Editorial:** RAII binds the `FILE*` handle to object lifetime: the constructor acquires it and the destructor releases it, so the file is closed automatically however the scope exits, including via an exception. The null check avoids calling `fclose` on a failed open. The drill teaches the core RAII resource-ownership idiom.
