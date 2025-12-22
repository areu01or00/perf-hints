---
name: performance-hints
description: This skill should be used when the user asks to "optimize code", "make this faster", "reduce allocations", "improve performance", "speed up", "performance bottleneck", "why is this slow", or mentions performance tuning. Provides the philosophy and techniques from Jeff Dean and Sanjay Ghemawat's Performance Hints.
---

This skill guides performance optimization using Jeff Dean and Sanjay Ghemawat's techniques from Google. For structured optimization workflows, use the `/perf` command.

## The Philosophy

Knuth is quoted out of context: *"premature optimization is the root of all evil."* The full quote:

> *"We should forget about small efficiencies, say about 97% of the time: premature optimization is the root of all evil. Yet we should not pass up our opportunities in that critical 3%."*

This skill is about that critical 3%. More importantly:

> *"In established engineering disciplines a 12% improvement, easily obtained, is never considered marginal; and I believe the same viewpoint should prevail in software engineering."*

**Don't dismiss small improvements.** Twenty separate 1% improvements compound significantly.

### Why "Write Simple Code and Optimize Later" Fails

1. **Flat profile problem**: Ignore performance everywhere, get no obvious hotspots—performance lost all over. No starting point for improvements.

2. **Library users suffer**: Slow library means users who can't easily fix it must understand your code and negotiate fixes.

3. **Heavy use constrains changes**: Harder to change systems in heavy use.

4. **Expensive workarounds**: Without knowing what's fixable, teams overprovision or over-replicate.

**Instead**: Choose the faster alternative when it doesn't significantly hurt readability.

## How to Think About Performance

- **Test code**: Worry about asymptotic complexity. Avoid slow tests—development cycle time matters.
- **Application code**: Is it initialization or hot path? That distinction is usually sufficient.
- **Library code**: Hard to predict future sensitivity. Follow simple techniques from the start—easier to find next bottleneck when profiling.

## Back-of-Envelope Estimation

Develop intuition by estimating:
1. Count operations of each type
2. Multiply by cost per operation
3. Add together for total cost

**If you don't know operation costs, you can't estimate.** Learn these:

```
L1 cache reference                             0.5 ns
Main memory reference                           50 ns
Datacenter round trip                       50,000 ns
Read 1MB from memory                        64,000 ns
Read 1MB from SSD                        1,000,000 ns
Disk seek                                5,000,000 ns
```

Track higher-level costs too: database reads, API latency, page render time.

## When Profiles Are Flat

No obvious hotspot? All low-hanging fruit picked? Then:

1. **Many small optimizations**: Twenty 1% improvements are significant
2. **Find loops higher in call stacks**: Flame graphs help. Restructure loops—build in one shot, not incrementally
3. **Replace overly general code**: Regex where prefix match suffices? Drop the regex.
4. **Reduce allocations**: Every allocation hits a different cache line. Attack highest allocation contributors.
5. **Hardware counters**: May reveal high cache miss rates

## Reference Files

For detailed techniques with code examples and benchmarks, consult:
- **`references/techniques.md`** - API design, memory layout, algorithmic improvements
- **`references/code-examples.md`** - Before/after code with actual benchmark results (10-50% improvements)
