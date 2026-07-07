## fact: Single Responsibility — one reason to change
tags: solid, srp, cohesion
track: design

The **Single Responsibility Principle** says a class should have exactly one reason to change — one *axis of change*, one stakeholder. A type that both renders a report and persists it changes when the format changes *and* when storage changes; those pressures pull it in different directions and eventually tangle.

Split along the axes. Each resulting type is smaller, testable in isolation, and reusable — you can swap the store without touching the renderer.

```cpp
// Two reasons to change fused into one class:
struct Report {
    std::string render() const;               // changes when the format changes
    void save(const std::string& path) const; // changes when storage changes
};

// One axis of change per type:
struct ReportRenderer { std::string render(const Report&) const; };
struct ReportStore    { void save(std::string_view text, const std::string& path); };
```
