## exercise: Scoped timer (RAII)
tags: raii, patterns

Write class `Timer`: constructor stores `std::chrono::steady_clock::now()` in member `start`; destructor computes `auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now() - start).count();` and prints it with `std::cout << ms << "ms\n";`.

hint: Measure a scope's duration by capturing the start in the constructor and the elapsed time in the destructor.
hint: Store `steady_clock::now()` on construction; subtract it from `now()` and report in the destructor.

```cpp
class Timer {
    std::chrono::steady_clock::time_point start;
public:
    Timer() : start(std::chrono::steady_clock::now()) {}
    ~Timer() {
        auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now() - start).count();
        std::cout << ms << "ms\n";
    }
};
```

```cpp
// harness
#include <chrono>
#include <iostream>
#include <sstream>
#include <cstdio>
//__USER__
int main() {
    std::ostringstream out;
    auto* old = std::cout.rdbuf(out.rdbuf());
    { Timer t; }
    std::cout.rdbuf(old);
    auto s = out.str();
    if (s.size() < 4 || s.substr(s.size() - 3) != "ms\n") return 1;
    std::puts("PASS");
}
```

**Editorial:** A scoped timer applies RAII to measurement: it records the start time on construction and, when it leaves scope, its destructor computes and prints the elapsed duration — so timing always ends exactly at scope exit, even on early return. `steady_clock` is the correct monotonic clock for intervals. The drill teaches RAII for automatic scope-bound actions.
