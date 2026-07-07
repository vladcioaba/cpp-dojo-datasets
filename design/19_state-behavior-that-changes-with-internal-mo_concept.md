## fact: State — behavior that changes with internal mode
tags: patterns, behavioral, state
track: design

When an object's response to the same call depends on its current mode, the naïve version is a `switch (state_)` repeated inside every method. The **State** pattern makes each state a *type* and delegates the call to the current state object; a transition is just swapping which state you hold. Adding a state is adding a class, not editing every method (Open/Closed again).

```cpp
struct Connection;
struct State { virtual void request(Connection&) = 0; virtual ~State() = default; };

struct Connection {
    std::unique_ptr<State> state;
    void request() { state->request(*this); }   // delegate to the current state
};

struct Closed : State { void request(Connection& c) override; };  // may set c.state = std::make_unique<Open>()
struct Open   : State { void request(Connection& c) override; };
```
