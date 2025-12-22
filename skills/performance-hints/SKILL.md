---
name: performance-hints
description: This skill should be used when the user asks to "optimize code", "make this faster", "reduce allocations", "improve performance", "speed up", "why is this slow", "review this code", "write tests", "design an API", "scale this", or mentions performance, memory, profiling, benchmarking, code review, testing, or engineering practices. Based on Google's Abseil engineering wisdom.
---

# Perf-Hints

Language-agnostic engineering wisdom from Google's [Abseil](https://abseil.io) - performance optimization, code review, testing, and software engineering practices.

## Philosophy

> "In established engineering disciplines a 12% improvement, easily obtained, is never considered marginal." — Donald Knuth

Don't dismiss small improvements. Twenty 1% improvements compound significantly.

### Why "Optimize Later" Fails

1. **Flat profile problem** - Performance lost everywhere, no obvious hotspot
2. **Library users suffer** - They can't easily fix your slow code
3. **Heavy use constrains changes** - Harder to change systems in production
4. **Expensive workarounds** - Teams overprovision instead of fixing

## Reference Routing

Consult the appropriate reference file based on the task:

| User asks about | Reference file |
|-----------------|----------------|
| Back-of-envelope, estimation, costs | `references/when-estimating.md` |
| Profiling, benchmarking, measurement | `references/when-measuring-performance.md` |
| Memory, allocations, cache, GC | `references/when-optimizing-memory.md` |
| "Why is this slow?", debugging perf | `references/when-debugging-perf.md` |
| Code review, PR review | `references/when-reviewing-code.md` |
| Testing, unit tests, mocking | `references/when-writing-tests.md` |
| API design, interfaces, libraries | `references/when-designing-apis.md` |
| Scaling, migrations, deprecation | `references/when-scaling.md` |
| Finding a specific article | `references/index.md` |

## Quick Reference

### Latency Numbers

```
L1 cache                    0.5 ns
Main memory                  50 ns
Datacenter round trip       50 μs
Read 1MB memory             64 μs
Read 1MB SSD                 1 ms
Disk seek                    5 ms
```

### Common Bottlenecks

| Issue | Pattern | Fix |
|-------|---------|-----|
| Sequential I/O | `for` + `await` | Parallelize |
| Client per request | `async with Client()` | Share client |
| N+1 queries | Loop of DB calls | Batch query |
| Hot loop allocations | Object in loop | Pre-allocate |

## Sources

All wisdom sourced from [abseil.io](https://abseil.io):
- [Performance Hints](https://abseil.io/fast/hints.html) - Core optimization guide
- [Fast Tips](https://abseil.io/fast/) - 25 performance articles
- [SWE Book](https://abseil.io/resources/swe-book) - Software Engineering at Google
- [C++ Tips](https://abseil.io/tips/) - Universal principles

For the full workflow with agents, use the `/perf` command.
