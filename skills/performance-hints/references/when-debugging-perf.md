# When Debugging Performance

Consult these Abseil sources when investigating "why is this slow?" or diagnosing performance issues.

## Start Here

- [Performance Hints](https://abseil.io/fast/hints.html) - The foundational debugging framework from Jeff Dean and Sanjay Ghemawat

## Diagnostic Methodology

- [How to estimate](https://abseil.io/fast/90) - Back-of-envelope to identify anomalies
- [Avoid sweeping street lights under rugs](https://abseil.io/fast/74) - Don't ignore what's easy to measure
- [Spooky action at a distance](https://abseil.io/fast/95) - Unexpected performance dependencies

## Common Culprits

- [Reducing memory indirections](https://abseil.io/fast/83) - Pointer chasing issues
- [Reducing memory bandwidth](https://abseil.io/fast/62) - Memory access bottlenecks
- [Improving regex efficiency](https://abseil.io/fast/21) - Regex as performance problem

## When Optimization Doesn't Work

- [Optimizations past their prime](https://abseil.io/fast/9) - When old optimizations become problems
- [Configuration knobs considered harmful](https://abseil.io/fast/52) - Complexity from tuning parameters
