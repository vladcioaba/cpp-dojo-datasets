## fact: Error handling — exceptions vs std::expected vs codes
tags: architecture, errors, exceptions, expected
track: design

Three tools, three jobs. **Exceptions**: rare, truly exceptional failures that most call sites can't handle locally — let them unwind to a place that can (keeps the happy path clean; costs only when thrown). **`std::expected<T,E>`** (C++23): *expected*, recoverable failures of a value-returning function, where you want the error visible in the type and no throwing on ordinary inputs. **Error codes**: at ABI boundaries, in embedded/`-fno-exceptions` builds, or hot paths where you can't pay for exceptions.

The anti-pattern is using exceptions for control flow you *expect* to hit on normal input (e.g. "parse failed" on user data). Make that expected-in-the-type.

```cpp
// Expected, recoverable failure — encode it, don't throw:
std::expected<Config, ParseError> parse(std::string_view text);   // C++23

auto cfg = parse(input);
if (!cfg) return report(cfg.error());   // explicit, no unwinding, checked at the call site
use(*cfg);
```
