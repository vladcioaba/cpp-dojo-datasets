## fact: Chain of Responsibility — pass it until someone handles it
tags: patterns, behavioral, chain-of-responsibility
track: design

**Chain of Responsibility** decouples the sender of a request from its handler by threading the request through a chain of handlers; each either handles it or passes it along. Middleware stacks (auth → rate-limit → logging → route) are exactly this. Handlers can be reordered or inserted without touching the others.

The base handler's default is "forward to next," so a concrete handler only writes the case it cares about.

```cpp
class Handler {
    std::unique_ptr<Handler> next_;
public:
    Handler* set_next(std::unique_ptr<Handler> n) { next_ = std::move(n); return next_.get(); }
    virtual void handle(Request& r) { if (next_) next_->handle(r); }   // default: pass along
    virtual ~Handler() = default;
};

class AuthHandler : public Handler {
    void handle(Request& r) override {
        if (!r.authed) { r.reject("401"); return; }   // stop the chain
        Handler::handle(r);                            // or defer to the next link
    }
};
```
