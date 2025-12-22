# Perf-Hints

A language-agnostic engineering wisdom plugin for Claude Code based on Google's [Abseil](https://abseil.io) - performance optimization, code review, testing, and software engineering practices.

## What It Does

Curated, task-organized knowledge from Google's engineering practices:
- **Performance optimization** - Back-of-envelope, profiling, memory
- **Code review** - Practices and checklists
- **Testing** - Strategies and principles
- **API design** - Performance-conscious interfaces
- **Scaling** - Large-scale changes and migrations

## Installation

```bash
/plugin marketplace add areu01or00/perf-hints
/plugin install perf-hints
```

## Usage

### Full Workflow

```bash
/perf
/perf backend/api/
```

The `/perf` command runs a 7-phase workflow:
1. **Scoping** - Understand what needs optimization
2. **Analysis** - Launch perf-analyzer agents
3. **Clarifying Questions** - Resolve ambiguities
4. **Fix Prioritization** - Rank by ROI
5. **Implementation** - Apply fixes
6. **Validation** - Launch perf-reviewer agents
7. **Summary** - Document changes

### Auto-Triggered Skill

Also triggers on:
- "optimize this code" / "make this faster"
- "why is this slow" / "review this code"
- "write tests" / "design an API"

## Reference Files

Task-based routing to curated links:

| Task | Reference |
|------|-----------|
| Estimation | `when-estimating.md` |
| Profiling | `when-measuring-performance.md` |
| Memory | `when-optimizing-memory.md` |
| Debugging perf | `when-debugging-perf.md` |
| Code review | `when-reviewing-code.md` |
| Testing | `when-writing-tests.md` |
| API design | `when-designing-apis.md` |
| Scaling | `when-scaling.md` |
| Full index | `index.md` |

## Sources

All from [abseil.io](https://abseil.io):
- [Performance Hints](https://abseil.io/fast/hints.html) - Core guide
- [Fast Tips](https://abseil.io/fast/) - 25 performance articles
- [SWE Book](https://abseil.io/resources/swe-book) - Software Engineering at Google
- [C++ Tips](https://abseil.io/tips/) - Universal principles (extracted)

## Components

| Component | Purpose |
|-----------|---------|
| `/perf` command | 7-phase optimization workflow |
| `perf-analyzer` agent | Back-of-envelope, bottleneck detection |
| `perf-reviewer` agent | Validates fixes for correctness |
| `performance-hints` skill | Philosophy + reference routing |

## Philosophy

> "In established engineering disciplines a 12% improvement, easily obtained, is never considered marginal." — Donald Knuth

## License

MIT
