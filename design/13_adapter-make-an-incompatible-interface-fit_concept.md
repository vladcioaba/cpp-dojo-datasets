## fact: Adapter — make an incompatible interface fit
tags: patterns, structural, adapter
track: design

Your code is written against interface A; a third-party class offers interface B that does the same job with different names. Rather than rewrite your code (or fork theirs), an **Adapter** implements A by delegating to a B it holds. It's the pattern behind `std::stack`, which adapts a container into a LIFO interface.

Prefer *object* adapters (hold the adaptee as a member) over inheriting from it — composition keeps you decoupled from the adaptee's implementation.

```cpp
// What your code expects:
struct Logger { virtual void log(std::string_view) = 0; virtual ~Logger() = default; };

// The legacy class you must reuse, with a mismatched API:
struct LegacyPrinter { void print_line(const char* s); };

// Adapter bridges the gap:
class PrinterLogger : public Logger {
    LegacyPrinter& p_;
public:
    explicit PrinterLogger(LegacyPrinter& p) : p_(p) {}
    void log(std::string_view m) override { p_.print_line(std::string(m).c_str()); }
};
```
