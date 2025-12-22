# When Designing APIs

Consult these Abseil sources for API design, interface design, and performance-conscious library design.

## Core Article

- [More Moore with better API design](https://abseil.io/fast/64) - Performance-first API design

## API Philosophy

- [Configuration knobs considered harmful](https://abseil.io/fast/52) - Why flags are bad
- [Two-way doors](https://abseil.io/fast/87) - Reversible design decisions
- [Optimizations past their prime](https://abseil.io/fast/9) - When old optimizations become problems

## SWE Book - API Evolution

- [Deprecation](https://abseil.io/resources/swe-book/html/ch20.html) - Removing old APIs safely
- [Dependency Management](https://abseil.io/resources/swe-book/html/ch27.html) - Managing API dependencies
- [Large-Scale Changes](https://abseil.io/resources/swe-book/html/ch28.html) - Evolving APIs at scale

## C++ Tips (Universal Principles)

- [Prefer factory functions](https://abseil.io/tips/42) - Object creation patterns
- [Avoid flags in library code](https://abseil.io/tips/45) - Library design
- [Avoid bool parameters](https://abseil.io/tips/94) - Readability at callsite
- [Flags are globals](https://abseil.io/tips/103) - Hidden dependencies
- [Option structs](https://abseil.io/tips/173) - Many-parameter functions
- [Returns over output params](https://abseil.io/tips/176) - Clearer signatures
