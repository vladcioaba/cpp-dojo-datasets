## challenge: fix: the deadlock fix that loses money
tags: concurrency, deadlock, debugging, code-review
track: core
difficulty: hard

This code review found a bug with history. Version 1 of `transfer()` locked `from.m` then `to.m` — and froze in production when two threads transferred in opposite directions: the classic ABBA deadlock. Someone "fixed" it by never holding both locks at once. The freeze is gone, but now the nightly reconciliation fails: money appears and disappears. Fix `transfer()` so it is *both* deadlock-free and atomic. `fraud_check()` must remain in the flow, and the signature stays.

hint: Follow `newFrom`: it is computed from a balance read under the lock, but written back MUCH later, after the lock was dropped and retaken. What did the other thread do to from.balance in between?
hint: The write-back `from.balance = newFrom` clobbers any concurrent deposit into `from` — a lost update. Releasing the lock mid-transaction traded a deadlock for silent corruption; the transfer must hold both mutexes across the whole thing.
hint: `std::scoped_lock lock(from.m, to.m);` acquires both mutexes deadlock-free (std::lock algorithm), no matter that the other thread names them in the opposite order. Do the debit, the fraud_check, and the credit all under it.

```cpp
// starter
struct Account {
    std::mutex m;
    long balance = 0;
};

// v1 locked from.m then to.m and DEADLOCKED under opposite-direction
// transfers (ABBA). v2 (below) never holds both locks... but the
// books no longer balance. Make it deadlock-free AND atomic.
void transfer(Account& from, Account& to, long amount) {
    from.m.lock();
    long newFrom = from.balance - amount;   // decide the new balance
    from.m.unlock();                        // drop the lock "to be safe"

    fraud_check();                          // other transfers run here

    to.m.lock();
    to.balance += amount;                   // credit
    to.m.unlock();

    from.m.lock();
    from.balance = newFrom;                 // write back a stale snapshot!
    from.m.unlock();
}
```

```cpp
struct Account {
    std::mutex m;
    long balance = 0;
};

void transfer(Account& from, Account& to, long amount) {
    std::scoped_lock lock(from.m, to.m);    // both locks, ABBA-proof
    long newFrom = from.balance - amount;
    fraud_check();                          // safe: still exclusive
    to.balance += amount;
    from.balance = newFrom;                 // snapshot can't go stale now
}
```

```cpp
// harness
#include <bits/stdc++.h>
void fraud_check() { std::this_thread::yield(); }  // other transfers run here
//__USER__
int main() {
    Account a, b;
    a.balance = 1000;
    b.balance = 1000;
    std::thread t1([&] { for (int i = 0; i < 1500; ++i) transfer(a, b, 1); });
    std::thread t2([&] { for (int i = 0; i < 1500; ++i) transfer(b, a, 1); });
    t1.join();
    t2.join();
    assert(a.balance + b.balance == 2000);      // no money created or destroyed
    assert(a.balance == 1000 && b.balance == 1000);
    std::puts("PASS");
}
```

**Editorial:** Two bugs, one lesson. The original ABBA deadlock — T1 holds `a.m` wanting `b.m`, T2 holds `b.m` wanting `a.m` — is a cycle in the lock graph, and the cure is to acquire both locks as one operation: `std::scoped_lock(from.m, to.m)` runs the `std::lock` try-and-back-off algorithm, which cannot cycle regardless of argument order (a fixed global order, e.g. by mutex address, works too). The "fix" under review made the opposite trade: by releasing `from.m` between the read and the write-back, it turned the transfer into a stale-snapshot lost update — the other thread's deposits into `from` land in the gap and are overwritten. Wrong answers to deadlocks are usually *less* locking; the right answer is *structured* locking: both mutexes, one acquisition, held for the whole invariant.
