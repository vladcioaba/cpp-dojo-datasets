## fact: The Observer pattern's modern problem is lifetime, not dispatch
tags: patterns, observer

Classic Observer: subject holds raw pointers to observers and calls `notify()`. The 2020s version: subject holds `std::vector<std::function<void(Event)>>` for dispatch — but the real design question is unsubscription. If an observer dies while subscribed, notify calls into freed memory.

Common answers: token-based unsubscribe, `std::weak_ptr` per observer checked at notify time, or a signals library (Boost.Signals2, Qt).
