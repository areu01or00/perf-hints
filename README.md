# Perf-Hints

A language agnostic Performance optimization workflow for Claude Code based on Jeff Dean and Sanjay Ghemawat's [Performance Hints](https://abseil.io/fast/hints.html) from Google.

## What It Does

Provides a systematic approach to performance optimization:
- Back-of-envelope calculations before touching code
- Bottleneck detection with estimated savings
- Validated fixes that don't break correctness

## Installation

```bash
/plugin marketplace add areu01or00/perf-hints
/plugin install perf-hints
```

## Usage

### Full Workflow (Recommended)

```bash
/perf
```

Or target specific code:

```bash
/perf backend/api/
```

The `/perf` command runs a 7-phase workflow:

1. **Scoping** - Understand what needs optimization
2. **Analysis** - Launch perf-analyzer agents for back-of-envelope calculations
3. **Clarifying Questions** - Resolve ambiguities before implementing
4. **Fix Prioritization** - Rank fixes by ROI
5. **Implementation** - Apply approved fixes
6. **Validation** - Launch perf-reviewer agents to verify correctness
7. **Summary** - Document changes and improvements

### Auto-Triggered Skill

The skill also auto-triggers when you ask things like:
- "optimize this code"
- "make this faster"
- "why is this slow"
- "reduce allocations"

## Components

### Agents

| Agent | Purpose |
|-------|---------|
| `perf-analyzer` | Traces bottlenecks, does back-of-envelope calculations, identifies optimization opportunities |
| `perf-reviewer` | Validates that optimizations are correct and actually deliver expected improvements |

### Skill

The `performance-hints` skill provides the philosophy and mental models for thinking about performance, including:
- Why "write simple code and optimize later" often fails
- Back-of-envelope estimation techniques
- What to do when profiles are flat
- Latency numbers every programmer should know

## Philosophy

> "In established engineering disciplines a 12% improvement, easily obtained, is never considered marginal; and I believe the same viewpoint should prevail in software engineering." — Donald Knuth

Don't dismiss small improvements. Twenty separate 1% improvements compound significantly.

## References

- [Performance Hints (abseil.io)](https://abseil.io/fast/hints.html) - Original source material
- [Latency Numbers Every Programmer Should Know](https://gist.github.com/jboner/2841832)

## License

MIT
