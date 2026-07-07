## fact: RAII is the pattern behind every other C++ pattern
tags: raii, core

Resource Acquisition Is Initialization: tie a resource's lifetime to an object's lifetime. Acquire in the constructor, release in the destructor. The compiler then guarantees cleanup on every exit path — returns, exceptions, early breaks.

`std::lock_guard`, `std::unique_ptr`, `std::fstream`, `std::jthread` — all RAII. If you write `new`/`delete` or `lock()`/`unlock()` by hand in application code, you are usually reinventing a worse version of it.

```cpp
{
    std::lock_guard<std::mutex> lk(m); // acquired
    do_work();                          // may throw — fine
}                                       // released, always
```
