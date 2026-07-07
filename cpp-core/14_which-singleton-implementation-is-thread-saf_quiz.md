## quiz: Which singleton implementation is thread-safe without locks?
tags: patterns, singleton

- [ ] `static Config* p = nullptr; if (!p) p = new Config;` inside `instance()`
- [x] `static Config cfg;` inside `instance()`, returning a reference
- [ ] A global `Config cfg;` at namespace scope, returned by `instance()`
- [ ] Double-checked locking with a plain `bool` flag

> Function-local statics get thread-safe initialization guaranteed by the standard since C++11 (Meyers Singleton). The lazy-pointer version races; the namespace-scope global has initialization-order problems across translation units; DCL with a plain bool is a data race.
