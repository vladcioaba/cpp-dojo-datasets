## fact: PIMPL — the compile-time firewall
tags: patterns, pimpl

Pointer to IMPLementation: the header exposes only a forward-declared `struct Impl;` and a `std::unique_ptr<Impl>`. All members live in the .cpp file. Changing private members no longer recompiles every file that includes your header.

Gotcha: the destructor must be defined in the .cpp (`Widget::~Widget() = default;`), because `unique_ptr<Impl>` needs the complete type to delete it.
