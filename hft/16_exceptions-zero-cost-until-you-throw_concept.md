## fact: Exceptions — zero-cost until you throw
tags: exceptions, hot-path, rtti
track: hft

Modern C++ uses **table-based ("zero-cost") exceptions**: the happy path has no per-call setup, so `try` blocks don't slow normal execution. The cost is the **throw**: unwinding walks tables, runs destructors, and can take microseconds — an eternity on the hot path, and non-deterministic.

HFT keeps throwing out of the trading loop: error codes, `std::optional`/`std::expected`, or `[[noreturn]]` fatal handlers. Throwing is fine for startup/config errors that happen once. The related trap: exception tables and **RTTI** (`dynamic_cast`, `typeid`) add binary size and hurt I-cache locality, which is why some shops build the hot library with `-fno-exceptions -fno-rtti`.
