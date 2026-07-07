## quiz: What does this print?
tags: core, containers

```cpp
std::map<std::string, int> m;
m["a"];
if (m["b"] == 1) {}
std::cout << m.size();
```

- [ ] 0
- [ ] 1
- [x] 2
- [ ] Undefined behavior

> `operator[]` on a map **inserts** a value-initialized element when the key is missing — even in a read-looking expression. Both `m["a"]` and `m["b"]` insert (values 0). Use `.find()`, `.at()`, `.contains()` (C++20) to look up without inserting.
