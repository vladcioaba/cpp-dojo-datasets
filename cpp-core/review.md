## quiz: review: the welcome email
tags: code-review, move-semantics
track: core
difficulty: easy

You're reviewing this code. What's the bug?

```cpp
void sendEmail(const std::string& to, const std::string& body);

void registerUser(std::vector<std::string>& audit, std::string email) {
    std::string body = "Welcome aboard, " + email + "!";
    audit.push_back(std::move(email));
    sendEmail(email, body);
}
```

- [ ] `audit.push_back` stores a reference to a local variable that dangles when the function returns
- [x] `email` is read after being moved from, so the mail is sent to a moved-from (in practice empty) address
- [ ] `body` must be passed to `sendEmail` by value because the reference parameter outlives the call
- [ ] calling `std::move` on a by-value function parameter is undefined behavior

> After `std::move(email)` hands the string to `push_back`, `email` is left in a valid-but-unspecified state — empty on every mainstream implementation — so `sendEmail` targets a blank address. Reorder the code to use `email` before moving it, or move it only in the last use. A reviewer should treat any read of a name after `std::move(name)` as a red flag.

## quiz: review: purging expired sessions
tags: code-review, iterators
track: core
difficulty: medium

You're reviewing this code. What's the bug?

```cpp
struct Session {
    int id;
    bool expired;
};

void purgeExpired(std::vector<Session>& sessions) {
    for (auto it = sessions.begin(); it != sessions.end(); ++it) {
        if (it->expired) {
            sessions.erase(it);
        }
    }
}
```

- [ ] `sessions.end()` must be cached before the loop because calling it repeatedly is invalid
- [ ] `erase` on a `std::vector` is O(n), so this function should use `std::list` instead
- [x] `erase(it)` invalidates `it`, so the `++it` that follows is undefined behavior and adjacent expired sessions get skipped
- [ ] the loop needs a `const_iterator` because `erase` requires one since C++11

> `vector::erase` invalidates the erased iterator and everything after it; incrementing it afterwards is UB, and even when it "works" the element shifted into the erased slot is never examined. The fix is `it = sessions.erase(it)` and incrementing only in the else-branch — or the erase-remove idiom. Any loop that erases from the container it is iterating deserves a hard look in review.

## quiz: review: printing a label
tags: code-review, lifetime
track: core
difficulty: medium

You're reviewing this code. What's the bug?

```cpp
std::string makeLabel(int id) {
    return "item-" + std::to_string(id);
}

void printLabel(int id) {
    std::string_view label = makeLabel(id);
    std::cout << "label=" << label << '\n';
}
```

- [ ] `std::string_view` is not null-terminated, so streaming it to `std::cout` reads past the end
- [ ] `makeLabel` returns by value, which forces an extra heap allocation per call
- [x] `label` views a temporary `std::string` that is destroyed at the end of the declaration, so the print reads freed memory
- [ ] `std::string_view` cannot be constructed from a `std::string` without calling `.data()` explicitly

> The temporary returned by `makeLabel` lives only until the end of the full expression that initializes `label`; the view dangles on the very next line. Store the result in a `std::string` (or print the temporary directly). In review, any `string_view` initialized from a function returning `std::string` by value is a lifetime bug.

## quiz: review: running a download task
tags: code-review, polymorphism
track: core
difficulty: medium

You're reviewing this code. What's the bug?

```cpp
struct Task {
    virtual void run() = 0;
};

struct DownloadTask : Task {
    std::string url;
    std::vector<char> buffer;
    void run() override { buffer.resize(1 << 20); }
};

void execute() {
    std::unique_ptr<Task> task = std::make_unique<DownloadTask>();
    task->run();
}
```

- [ ] `run` is declared `override` but the base method is pure virtual, so the override never binds
- [x] `Task` has no virtual destructor, so destroying `DownloadTask` through `unique_ptr<Task>` is undefined behavior and leaks `url` and `buffer` in practice
- [ ] `std::make_unique<DownloadTask>()` cannot be assigned to `std::unique_ptr<Task>` without an explicit cast
- [ ] `buffer.resize` inside `run` reallocates the vector and invalidates the `task` pointer

> When `task` goes out of scope, `unique_ptr<Task>` runs `delete` on a `Task*` whose static type has a non-virtual destructor — undefined behavior, and typically the derived members' destructors never run, leaking the buffer. Add `virtual ~Task() = default;` to the base. Every polymorphic base deleted through a base pointer needs a virtual destructor; reviewers should check this on any `virtual` function they see.

## quiz: review: parallel match counting
tags: code-review, concurrency
track: core
difficulty: hard

You're reviewing this code. What's the bug?

```cpp
int matches = 0;

void countMatches(const std::vector<int>& data, int needle) {
    std::vector<std::thread> workers;
    for (int t = 0; t < 4; ++t) {
        workers.emplace_back([&, t] {
            for (std::size_t i = t; i < data.size(); i += 4) {
                if (data[i] == needle) ++matches;
            }
        });
    }
    for (auto& w : workers) w.join();
}
```

- [ ] the lambda captures `data` by reference, which dangles because the threads outlive `countMatches`
- [ ] `emplace_back` copies the `std::thread`, and threads are not copyable
- [x] four threads increment `matches` with no synchronization — a data race, so counts are lost and the behavior is undefined
- [ ] the stride `i += 4` skips elements whenever `data.size()` is not a multiple of 4

> `++matches` is a read-modify-write on a shared non-atomic int from four threads at once: a data race, which is undefined behavior and in practice loses increments. Make it `std::atomic<int>`, or better, accumulate per-thread locals and sum after `join`. The reference captures are fine here because every worker is joined before the function returns.

## quiz: review: the money transfer
tags: code-review, concurrency
track: core
difficulty: hard

You're reviewing this code. What's the bug?

```cpp
struct Account {
    std::mutex m;
    long balance = 0;
};

void transfer(Account& from, Account& to, long amount) {
    std::lock_guard<std::mutex> lockFrom(from.m);
    std::lock_guard<std::mutex> lockTo(to.m);
    from.balance -= amount;
    to.balance += amount;
}
// thread A: transfer(checking, savings, 100);
// thread B: transfer(savings, checking, 50);
```

- [ ] `std::lock_guard` does not release the mutex if an exception is thrown between the two locks
- [x] the two calls lock the same pair of mutexes in opposite order, so each thread can hold one mutex while waiting for the other — deadlock
- [ ] `balance` must be `std::atomic<long>` even while both mutexes are held
- [ ] `transfer` modifies `from.balance` without checking for a negative result first

> Thread A locks `checking.m` then wants `savings.m`; thread B locks `savings.m` then wants `checking.m` — a classic ABBA deadlock. Acquire both with one deadlock-avoiding operation, `std::scoped_lock lock(from.m, to.m);`, or lock in a globally consistent order (e.g. by address). Whenever review shows two mutexes taken one after the other, ask who else takes them in the other order.

## quiz: review: appending a summary line
tags: code-review, lifetime
track: core
difficulty: medium

You're reviewing this code. What's the bug?

```cpp
void appendSummary(std::vector<std::string>& lines) {
    if (lines.empty()) {
        return;
    }
    const std::string& title = lines.front();
    lines.push_back("----------");
    lines.push_back("End of: " + title);
}
```

- [ ] `lines.front()` returns a temporary, so binding it to a reference extends the wrong lifetime
- [x] `push_back` can reallocate the vector, leaving `title` a dangling reference when it is read on the last line
- [ ] the second `push_back` invalidates the string literal appended by the first one
- [ ] `"End of: " + title` concatenates a `const char*` with a `std::string`, which does not compile

> If the first `push_back` grows the vector past its capacity, every element is moved to new storage and `title` — a reference into the old buffer — dangles; the concatenation then reads freed memory. Take a copy (`std::string title = lines.front();`) before mutating the vector. References, pointers, and iterators into a `std::vector` are only valid until the next capacity-changing operation.

## quiz: review: adjacent duplicates
tags: code-review, integer-arithmetic
track: core
difficulty: medium

You're reviewing this code. What's the bug?

```cpp
bool hasAdjacentDuplicate(const std::vector<int>& values) {
    for (std::size_t i = 0; i < values.size() - 1; ++i) {
        if (values[i] == values[i + 1]) {
            return true;
        }
    }
    return false;
}
```

- [ ] `i + 1` overflows `std::size_t` on the last iteration and wraps to zero
- [ ] the function returns after the first pair, so later duplicates are never reported
- [x] when `values` is empty, `values.size() - 1` wraps to a huge unsigned value and the loop reads far out of bounds
- [ ] `std::size_t` cannot index a `std::vector<int>`; the index must be `std::vector<int>::size_type`

> `size()` returns an unsigned type, so `0 - 1` wraps to `SIZE_MAX` and an empty input turns the loop bound into effectively infinity — out-of-bounds reads and a likely crash. Guard with `if (values.size() < 2) return false;` or write the condition as `i + 1 < values.size()`, which never underflows. Unsigned subtraction in loop bounds is a classic review catch.

## quiz: review: moving average
tags: code-review, bounds
track: core
difficulty: easy

You're reviewing this code. What's the bug?

```cpp
double movingAverage(const double* samples, int count) {
    double sum = 0.0;
    for (int i = 0; i <= count; ++i) {
        sum += samples[i];
    }
    return sum / count;
}
```

- [ ] `sum / count` performs integer division and truncates the result
- [x] the loop condition `i <= count` reads `samples[count]`, one element past the end of the array
- [ ] `sum` should be initialized to `samples[0]`, not `0.0`
- [ ] `count` should be `std::size_t`, because a negative count makes the loop skip all elements

> With `i <= count` the final iteration reads `samples[count]`, which is one past the last valid element — an out-of-bounds read that silently corrupts the average (or crashes). The condition must be `i < count`. Off-by-one on `<=` versus `<` at an array bound is the first thing to check in any raw-pointer loop.

## quiz: review: config lookup
tags: code-review, null-safety
track: core
difficulty: easy

You're reviewing this code. What's the bug?

```cpp
struct Config {
    int timeoutMs;
};

Config* findConfig(const std::string& name);   // returns nullptr when absent

int getTimeout(const std::string& name) {
    Config* cfg = findConfig(name);
    return cfg->timeoutMs;
}
```

- [ ] `findConfig` returns a raw pointer, so the caller of `getTimeout` leaks a `Config` on every call
- [ ] `name` is passed by const reference and may dangle inside `findConfig`
- [x] `cfg` is dereferenced without a null check, so an unknown name dereferences `nullptr`
- [ ] `timeoutMs` is uninitialized in `Config`, so the returned value is garbage even on success

> The comment on `findConfig` documents that it can return `nullptr`, and `getTimeout` dereferences the result unconditionally — undefined behavior (usually a crash) for any unknown name. Check the pointer and return a sensible default or signal the error. When an API documents a null return, every call site must be reviewed for a matching check.

## quiz: review: retry policy defaults
tags: code-review, classes
track: core
difficulty: easy

You're reviewing this code. What's the bug?

```cpp
class RetryPolicy {
public:
    RetryPolicy(int maxAttempts, int delayMs) {
        maxAttempts = maxAttempts;
        delayMs = delayMs;
    }
    int attempts() const { return maxAttempts; }
    int delay() const { return delayMs; }
private:
    int maxAttempts = 3;
    int delayMs = 100;
};
```

- [ ] the members are initialized twice — once by the default initializers and once in the constructor body
- [x] the constructor parameters shadow the members, so each statement assigns a parameter to itself and every policy keeps the defaults 3 and 100
- [ ] the constructor is missing `explicit`, so a stray brace-list converts to `RetryPolicy` silently
- [ ] `attempts()` and `delay()` return copies instead of const references, losing updates

> Inside the constructor body, `maxAttempts` names the parameter, not the member, so `maxAttempts = maxAttempts;` is a self-assignment and the members silently keep their default values. Use a member-initializer list (`: maxAttempts(maxAttempts), delayMs(delayMs)`) or `this->maxAttempts = maxAttempts;`. Same-named constructor parameters are fine — but only with an initializer list.

## quiz: review: ramp to target
tags: code-review, floating-point
track: core
difficulty: easy

You're reviewing this code. What's the bug?

```cpp
bool rampReaches(double start, double step, double target, int maxSteps) {
    double level = start;
    for (int i = 0; i < maxSteps; ++i) {
        if (level == target) {
            return true;
        }
        level += step;
    }
    return false;
}
// rampReaches(0.0, 0.1, 0.3, 100) is expected to return true
```

- [ ] `level += step` accumulates into a `double` but `step` is promoted to `long double`, changing the sum
- [x] `level == target` compares accumulated floating-point values exactly; 0.1 + 0.1 + 0.1 is not exactly 0.3, so the function returns false
- [ ] the loop runs `maxSteps` times even after passing `target`, wasting iterations
- [ ] `start`, `step`, and `target` should be `float`, since `double` cannot represent 0.1

> 0.1 has no exact binary representation, so three accumulated steps land at 0.30000000000000004 — never exactly equal to `0.3` — and the ramp "misses" its target. Compare with a tolerance (`std::fabs(level - target) < eps`) or with `level >= target` for a monotone ramp. Exact `==` between computed floating-point values is almost always wrong.

## quiz: review: block offsets
tags: code-review, integer-arithmetic
track: core
difficulty: medium

You're reviewing this code. What's the bug?

```cpp
constexpr int kBlockSize = 64 * 1024;   // 64 KiB blocks

long long blockOffset(int blockIndex) {
    return blockIndex * kBlockSize;
}

bool validOffset(int blockIndex, long long fileSize) {
    return blockOffset(blockIndex) + kBlockSize <= fileSize;
}
```

- [ ] `kBlockSize` participates in a `long long` expression, so it must be declared `long long` to compile
- [ ] `<=` in `validOffset` should be `<`, because a block ending exactly at `fileSize` is invalid
- [x] `blockIndex * kBlockSize` multiplies two `int`s, so the product overflows before it is widened to `long long` — offsets go wrong past 2 GiB
- [ ] returning `long long` from an `int` expression truncates the upper 32 bits

> The multiplication happens entirely in `int`; for `blockIndex >= 32768` the product exceeds `INT_MAX`, which is signed overflow — undefined behavior that in practice yields a negative offset. Widen an operand first: `return static_cast<long long>(blockIndex) * kBlockSize;`. The `long long` return type does not help — the damage is done before the conversion. Any `int * int` assigned to a 64-bit type deserves scrutiny.
