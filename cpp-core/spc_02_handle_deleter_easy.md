## challenge: unique_ptr for a C-style handle
tags: smart-pointers, raii
track: core
difficulty: easy

A legacy C-style API hands out `DataFile*` handles that must be released with `df_close` — never plain `delete`. Wrap it: make `FileHandle` a `std::unique_ptr` that calls `df_close` automatically, and keep it pointer-sized (a stateless deleter, not a function pointer). The harness verifies `df_close` runs exactly once per open, at the right moments.

hint: `std::unique_ptr<T, D>` takes the deleter as a second template parameter; the default one calls `delete`, which skips `df_close` entirely.
hint: A function pointer deleter (`void(*)(DataFile*)`) works but doubles `sizeof(FileHandle)` — the pointer must be stored. The harness checks the size.
hint: Write a stateless functor: `struct DfCloser { void operator()(DataFile* f) const { df_close(f); } };` then `using FileHandle = std::unique_ptr<DataFile, DfCloser>;` — empty deleters take no space.

```cpp
// starter
// Legacy C-style API: every df_open must be paired with exactly one df_close.
struct DataFile { int id; };
inline int g_open_count = 0;
inline int g_close_count = 0;

DataFile* df_open(int id) { ++g_open_count; return new DataFile{id}; }
int df_read(const DataFile* f) { return f->id * 10; }
void df_close(DataFile* f) { ++g_close_count; delete f; }

// TODO: FileHandle must call df_close (not plain delete) when it dies,
// and stay the size of a raw pointer. This alias is wrong on both counts:
using FileHandle = std::unique_ptr<DataFile>;

FileHandle open_file(int id) {
    return FileHandle(df_open(id));
}
```

```cpp
// Legacy C-style API: every df_open must be paired with exactly one df_close.
struct DataFile { int id; };
inline int g_open_count = 0;
inline int g_close_count = 0;

DataFile* df_open(int id) { ++g_open_count; return new DataFile{id}; }
int df_read(const DataFile* f) { return f->id * 10; }
void df_close(DataFile* f) { ++g_close_count; delete f; }

// Stateless functor deleter: no storage cost, impossible to forget.
struct DfCloser {
    void operator()(DataFile* f) const { df_close(f); }
};
using FileHandle = std::unique_ptr<DataFile, DfCloser>;

FileHandle open_file(int id) {
    return FileHandle(df_open(id));
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    {
        FileHandle f = open_file(7);
        assert(f != nullptr);
        assert(g_open_count == 1);
        assert(df_read(f.get()) == 70);     // borrow via get(), no transfer
        assert(g_close_count == 0);

        FileHandle g = std::move(f);        // ownership moves; still no close
        assert(!f);
        assert(g_close_count == 0);
        assert(df_read(g.get()) == 70);
    }                                       // g dies -> df_close, exactly once
    assert(g_open_count == 1);
    assert(g_close_count == 1);

    {
        FileHandle a = open_file(1);
        FileHandle b = open_file(2);
        a = std::move(b);                   // a's old file closed immediately
        assert(g_close_count == 2);
    }                                       // remaining handle closed
    assert(g_open_count == 3);
    assert(g_close_count == 3);

    // Stateless functor deleter keeps the handle pointer-sized.
    static_assert(sizeof(FileHandle) == sizeof(DataFile*),
                  "use a stateless deleter, not a function pointer");
    std::puts("PASS");
}
```

**Editorial:** The starter's `std::unique_ptr<DataFile>` uses the default deleter — plain `delete` — so memory is reclaimed but `df_close` never runs: the close counter stays at zero, which in real code means leaked descriptors, unflushed buffers, or a resource the library thinks is still checked out. The deleter is part of `unique_ptr`'s *type*, supplied as the second template parameter. A function pointer (`std::unique_ptr<DataFile, void(*)(DataFile*)>`) works but must be stored per-handle, doubling the size and allowing a wrong function to be passed at each construction site. The stateless functor is the idiomatic form: `DfCloser` is an empty type, so the empty-base optimization inside `unique_ptr` makes the handle exactly pointer-sized, and the correct release function is welded into the type — no construction site can get it wrong. This pattern (`FILE*`/`fclose`, `SSL*`/`SSL_free`, `sqlite3*`/`sqlite3_close`) is the standard way to give any C API automatic, exception-safe cleanup. Note what the harness demonstrates along the way: moves transfer the obligation without triggering it, and move-assignment closes the destination's old handle immediately.
