## exercise: Scoped timer (RAII)
tags: raii, patterns

Write class `Timer`: constructor stores `std::chrono::steady_clock::now()` in member `start`; destructor computes `auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::steady_clock::now() - start).count();` and prints it with `std::cout << ms << "ms\n";`.

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
