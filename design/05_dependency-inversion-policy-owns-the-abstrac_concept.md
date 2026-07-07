## fact: Dependency Inversion — policy owns the abstraction
tags: solid, dip, abstractions
track: design

The **Dependency Inversion Principle**: high-level policy should not depend on low-level details; both depend on an abstraction — and the abstraction belongs to the high-level module. `TokenService` needs "current time," so *it* defines a `Clock` interface. `SystemClock` and `FakeClock` implement it. The dependency arrow now points *toward* the policy, not away from it.

This is what makes the core testable: inject a `FakeClock` and expiry logic becomes deterministic, no sleeping, no wall clock.

```cpp
struct Clock {   // abstraction owned by the high-level module
    virtual std::chrono::system_clock::time_point now() const = 0;
    virtual ~Clock() = default;
};

class TokenService {                 // policy — knows nothing of which clock
    const Clock& clock_;
public:
    explicit TokenService(const Clock& c) : clock_(c) {}
    bool expired(std::chrono::system_clock::time_point deadline) const {
        return clock_.now() > deadline;
    }
};
```
