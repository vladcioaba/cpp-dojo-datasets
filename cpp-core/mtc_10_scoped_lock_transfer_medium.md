## challenge: two-mutex transfer without deadlock
tags: concurrency, deadlock, mutex
track: core
difficulty: medium

Each `Account` carries its own mutex. Implement `transfer()` so the debit and credit happen atomically — both mutexes held at once. The trap: the harness runs `transfer(a, b, ...)` and `transfer(b, a, ...)` concurrently, so locking "first argument, then second argument" is the classic ABBA deadlock. Use the tool built for exactly this.

hint: Locking from.m then to.m deadlocks when another thread calls transfer with the arguments swapped: each thread holds one mutex and waits for the other, forever.
hint: `std::scoped_lock lock(from.m, to.m);` locks any number of mutexes via the std::lock deadlock-avoidance algorithm — safe regardless of argument order — and releases both on scope exit.
hint: With both locks held, the body is just `from.balance -= amount; to.balance += amount;`. (The manual alternative — locking in a globally consistent order, e.g. by address — also works, but scoped_lock is the one-liner.)

```cpp
// starter
struct Account {
    std::mutex m;
    long balance = 0;
};

// Move `amount` from one account to the other, atomically.
// transfer(a, b, ...) and transfer(b, a, ...) run at the same time —
// your locking must not deadlock in either direction.
void transfer(Account& from, Account& to, long amount) {
    // TODO: take BOTH mutexes with one std::scoped_lock,
    //       then move the money.
}
```

```cpp
struct Account {
    std::mutex m;
    long balance = 0;
};

void transfer(Account& from, Account& to, long amount) {
    std::scoped_lock lock(from.m, to.m);  // both at once — ABBA-proof
    from.balance -= amount;
    to.balance   += amount;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Account a, b;
    a.balance = 6000;
    b.balance = 4000;
    transfer(a, b, 1000);                       // single-threaded sanity
    assert(a.balance == 5000 && b.balance == 5000);

    std::thread t1([&] { for (int i = 0; i < 2000; ++i) transfer(a, b, 1); });
    std::thread t2([&] { for (int i = 0; i < 2000; ++i) transfer(b, a, 1); });
    t1.join();
    t2.join();
    assert(a.balance + b.balance == 10000);     // money conserved
    assert(a.balance == 5000 && b.balance == 5000);
    std::puts("PASS");
}
```

**Editorial:** Fine-grained locking (one mutex per account) scales better than one big bank mutex — but any operation that needs *two* locks must acquire them without creating a cycle. Lock in argument order and two opposing transfers each grab their first mutex, then wait on each other forever: the ABBA deadlock, the most-asked concurrency interview question in existence. `std::scoped_lock(m1, m2, ...)` delegates to `std::lock`, which acquires the set with a try-and-back-off algorithm guaranteeing no deadlock whatever order threads name the mutexes in. The manual discipline — impose one global order, such as `&from.m < &to.m` — also works and is worth being able to write, but the variadic `scoped_lock` is the modern one-liner reviewers want to see.
