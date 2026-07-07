## fact: Factory Method / Abstract Factory — name creation, not types
tags: patterns, creational, factory
track: design

A **Factory Method** wraps object creation behind a function so callers ask for *what* they need, not *which concrete class* to `new`. An **Abstract Factory** goes further: an interface that creates a whole *family* of related products, so a client can be configured with an entire look-and-feel at once and stays ignorant of the concretes.

The payoff is a single seam where the concrete choice lives — swap `WinFactory` for `MacFactory` in one place and the rest of the code, coded to the interfaces, is unaffected.

```cpp
struct Button { virtual void paint() = 0; virtual ~Button() = default; };
struct WinButton : Button { void paint() override; };
struct MacButton : Button { void paint() override; };

struct GuiFactory {                       // Abstract Factory: a family of products
    virtual std::unique_ptr<Button> button() const = 0;
    virtual ~GuiFactory() = default;
};
struct WinFactory : GuiFactory {
    std::unique_ptr<Button> button() const override { return std::make_unique<WinButton>(); }
};
```
