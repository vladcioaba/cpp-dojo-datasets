## fact: Proxy — same interface, controlled access
tags: patterns, structural, proxy
track: design

A **Proxy** implements the same interface as the real object but sits in front of it to control access: lazy creation (virtual proxy), caching, access checks, reference counting, or remoting. The client can't tell it isn't talking to the real thing.

Below, an expensive `RealImage` is only constructed on first `draw()` — the classic *virtual proxy* deferring heavy work until it's actually needed.

```cpp
struct Image { virtual void draw() = 0; virtual ~Image() = default; };
class RealImage : public Image {          // loads pixels from disk in its constructor
public: explicit RealImage(std::string path); void draw() override;
};

class LazyImage : public Image {          // virtual proxy: defer the expensive load
    std::string path_;
    std::unique_ptr<RealImage> real_;
public:
    explicit LazyImage(std::string p) : path_(std::move(p)) {}
    void draw() override {
        if (!real_) real_ = std::make_unique<RealImage>(path_);   // create on first use
        real_->draw();
    }
};
```
