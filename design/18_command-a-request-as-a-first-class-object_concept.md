## fact: Command — a request as a first-class object
tags: patterns, behavioral, command, undo
track: design

The **Command** pattern packages "do this" into an object with an `execute()` (and often `undo()`). Once a request is an object you can queue it, log it, replay it, schedule it, or reverse it. An undo stack is nothing more than a `vector<unique_ptr<Command>>` you pop and call `undo()` on.

The command captures its own parameters and target, so the invoker (a menu, a scheduler) stays completely generic.

```cpp
struct Command { virtual void execute() = 0; virtual void undo() = 0; virtual ~Command() = default; };

class AddText : public Command {
    Document& doc_;
    std::string text_;
public:
    AddText(Document& d, std::string t) : doc_(d), text_(std::move(t)) {}
    void execute() override { doc_.append(text_); }
    void undo()    override { doc_.erase_last(text_.size()); }
};
// Undo history: std::vector<std::unique_ptr<Command>>.
```
