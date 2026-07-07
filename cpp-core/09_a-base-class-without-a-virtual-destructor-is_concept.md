## fact: A base class without a virtual destructor is a trap
tags: core, inheritance

Deleting a derived object through a base pointer is **undefined behavior** unless the base destructor is `virtual`. The derived destructor never runs; members leak.

Rule: a base class should have either a `virtual` destructor (polymorphic use) or a `protected` non-virtual one (interface-only use, no deletion through base).
