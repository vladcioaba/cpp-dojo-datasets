## fact: ABI stability and the firewall boundary
tags: architecture, abi, firewall, libraries
track: design

A shared library's **ABI** — struct layout, `sizeof`, member order, vtable shape, mangled symbol names — is a binary contract with already-compiled callers. Source that still *compiles* can still break binary compatibility: add a data member and `sizeof` changes; add or reorder a `virtual` and every caller's vtable offsets are wrong. Callers that weren't recompiled then corrupt memory.

Stable-ABI libraries hide layout behind a **firewall** — an opaque handle or pure-abstract interface — and only ever *add* new functions, never change existing signatures or the visible layout. (PIMPL, in the core track, is the in-language version of this firewall.)

```cpp
// ABI-fragile: callers bake in sizeof(Widget) and field offsets.
// ABI-stable boundary: an opaque handle whose layout callers can never see.
struct Widget;                                      // incomplete type — no size exposed
Widget* widget_create();
void    widget_set_title(Widget*, const char* utf8);// only ADD new functions over time
void    widget_destroy(Widget*);
```
