# cpp-dojo-datasets

Learning content for [cpp-dojo](https://github.com/vladcioaba/cpp-dojo) — the daily C++ mastery + FAANG/HFT interview trainer. Kept in its own repo so content ships independently of the app; the app loads it directly from this repo's GitHub raw. Added to the code repo as a git submodule at `datasets/`.

## Layout

One card per file, grouped by topic:

```
{topic}/{NN}_{headline}_{level}.md
```

| Topic | What |
|-------|------|
| `cpp-core` | general C++ facts, quizzes, write-drills, design-pattern classics |
| `hft` | low-latency C++ (cache, memory model, lock-free, latency) + timed challenges |
| `faang` | LeetCode-style DS&A coding problems (compile-checked) |
| `quant` | probability / EV / market-making brainteasers |
| `fpga` | FPGA-for-HFT (HDL, timing, pipelining, tick-to-trade) |
| `design` | OOP, SOLID, design patterns, software architecture — by example |
| `snippets` | code snippets captured from the wild, with analysis |

Each card is a markdown section headed `## {type}: {Title}` where type is `fact` / `quiz` / `exercise` / `challenge` / `snippet`, optionally with `tags:`, `track:`, `difficulty:` lines. See the code repo README for the per-type format.

## Generated files

- **`bundle.md`** — every card concatenated. The app fetches this one file (via the backend, which proxies it from GitHub raw). **Do not edit by hand.**
- **`manifest.json`** — index of all cards `{topic, file, type, track, level, title}`.

Regenerate both after any change:

```bash
node tools/build.js
```
