## fact: Data-oriented design — when OOP is the wrong tool
tags: architecture, dod, aos-soa, performance
track: design

OOP groups data by *object* — an **Array of Structs (AoS)**. But when a hot loop touches one field across millions of objects, each cache line drags in the other fields you don't read, wasting most of your memory bandwidth. **Data-oriented design** groups data by *field* — a **Struct of Arrays (SoA)** — so a loop streams exactly the bytes it needs, and the CPU can vectorize and prefetch it. This is often several times faster.

The lesson isn't "OOP is bad"; it's that encapsulating per-object data is the wrong default when throughput over bulk uniform data dominates. Design for the access pattern, not the noun.

```cpp
// AoS: updating position still pulls vx/vy/vz/mass into cache.
struct Particle { float x, y, z, vx, vy, vz, mass; };
std::vector<Particle> aos;

// SoA: the position update touches only three dense, contiguous arrays.
struct Particles {
    std::vector<float> x, y, z, vx, vy, vz, mass;
};
// for (std::size_t i = 0; i < n; ++i) ps.x[i] += ps.vx[i] * dt;  // vectorizes, cache-friendly
```
