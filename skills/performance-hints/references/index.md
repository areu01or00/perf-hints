# Abseil Wisdom Index

Master index of Google engineering wisdom from [abseil.io](https://abseil.io).

## Task-Based References

| Task | Reference File |
|------|----------------|
| Back-of-envelope, estimation | `when-estimating.md` |
| Profiling, benchmarking | `when-measuring-performance.md` |
| Memory, allocations, cache | `when-optimizing-memory.md` |
| "Why is this slow?" | `when-debugging-perf.md` |
| Code review | `when-reviewing-code.md` |
| Testing strategy | `when-writing-tests.md` |
| API/interface design | `when-designing-apis.md` |
| Large-scale changes | `when-scaling.md` |

---

## Complete Source Index

### Performance Hints (Core)

- [Performance Hints](https://abseil.io/fast/hints.html) - The foundational guide from Jeff Dean and Sanjay Ghemawat

### Fast Tips (25 Articles)

All articles: [abseil.io/fast](https://abseil.io/fast/)

| # | Title |
|---|-------|
| 7 | [Optimizing for application productivity](https://abseil.io/fast/7) |
| 9 | [Optimizations past their prime](https://abseil.io/fast/9) |
| 21 | [Improving regex efficiency](https://abseil.io/fast/21) |
| 26 | [Fixing things with hashtable profiling](https://abseil.io/fast/26) |
| 39 | [Beware microbenchmarks bearing gifts](https://abseil.io/fast/39) |
| 52 | [Configuration knobs considered harmful](https://abseil.io/fast/52) |
| 53 | [Hardware Performance Counters](https://abseil.io/fast/53) |
| 60 | [In-process profiling lessons](https://abseil.io/fast/60) |
| 62 | [Reducing memory bandwidth](https://abseil.io/fast/62) |
| 64 | [More Moore with better API design](https://abseil.io/fast/64) |
| 70 | [Defining optimization success](https://abseil.io/fast/70) |
| 72 | [Optimizing optimization](https://abseil.io/fast/72) |
| 74 | [Avoid sweeping street lights under rugs](https://abseil.io/fast/74) |
| 75 | [How to microbenchmark](https://abseil.io/fast/75) |
| 79 | [One tradeoff at a time](https://abseil.io/fast/79) |
| 83 | [Reducing memory indirections](https://abseil.io/fast/83) |
| 87 | [Two-way doors](https://abseil.io/fast/87) |
| 88 | [Avoid the jelly beans trap](https://abseil.io/fast/88) |
| 90 | [How to estimate](https://abseil.io/fast/90) |
| 93 | [Robots never sleep](https://abseil.io/fast/93) |
| 94 | [Decision making in data-imperfect world](https://abseil.io/fast/94) |
| 95 | [Spooky action at a distance](https://abseil.io/fast/95) |
| 97 | [Virtuous ecosystem cycles](https://abseil.io/fast/97) |
| 98 | [Measurement has an ROI](https://abseil.io/fast/98) |
| 99 | [llvm-mca](https://abseil.io/fast/99) |

### SWE Book (18 Chapters)

Full book: [abseil.io/resources/swe-book](https://abseil.io/resources/swe-book)

| Ch | Title |
|----|-------|
| 6 | [How to Work Well on Teams](https://abseil.io/resources/swe-book/html/ch06.html) |
| 7 | [Knowledge Sharing](https://abseil.io/resources/swe-book/html/ch07.html) |
| 11 | [Measuring Engineering Productivity](https://abseil.io/resources/swe-book/html/ch11.html) |
| 13 | [Style Guides and Rules](https://abseil.io/resources/swe-book/html/ch13.html) |
| 14 | [Code Review](https://abseil.io/resources/swe-book/html/ch14.html) |
| 15 | [Documentation](https://abseil.io/resources/swe-book/html/ch15.html) |
| 16 | [Testing Overview](https://abseil.io/resources/swe-book/html/ch16.html) |
| 17 | [Unit Testing](https://abseil.io/resources/swe-book/html/ch17.html) |
| 18 | [Test Doubles](https://abseil.io/resources/swe-book/html/ch18.html) |
| 19 | [Larger Testing](https://abseil.io/resources/swe-book/html/ch19.html) |
| 20 | [Deprecation](https://abseil.io/resources/swe-book/html/ch20.html) |
| 22 | [Version Control](https://abseil.io/resources/swe-book/html/ch22.html) |
| 24 | [Build Systems](https://abseil.io/resources/swe-book/html/ch24.html) |
| 26 | [Static Analysis](https://abseil.io/resources/swe-book/html/ch26.html) |
| 27 | [Dependency Management](https://abseil.io/resources/swe-book/html/ch27.html) |
| 28 | [Large-Scale Changes](https://abseil.io/resources/swe-book/html/ch28.html) |
| 29 | [Continuous Integration](https://abseil.io/resources/swe-book/html/ch29.html) |
| 30 | [Continuous Delivery](https://abseil.io/resources/swe-book/html/ch30.html) |

### C++ Tips (Universal Principles)

Selected tips with language-agnostic principles: [abseil.io/tips](https://abseil.io/tips/)

| # | Principle |
|---|-----------|
| 42 | [Prefer factory functions](https://abseil.io/tips/42) |
| 45 | [Avoid flags in library code](https://abseil.io/tips/45) |
| 77 | [Temporaries, moves, copies](https://abseil.io/tips/77) |
| 94 | [Avoid bool parameters](https://abseil.io/tips/94) |
| 103 | [Flags are globals](https://abseil.io/tips/103) |
| 112 | [emplace vs push_back](https://abseil.io/tips/112) |
| 117 | [Copy elision](https://abseil.io/tips/117) |
| 122 | [Test fixtures, clarity](https://abseil.io/tips/122) |
| 135 | [Test contract, not impl](https://abseil.io/tips/135) |
| 161 | [Good locals vs bad locals](https://abseil.io/tips/161) |
| 171 | [Avoid sentinel values](https://abseil.io/tips/171) |
| 173 | [Option structs](https://abseil.io/tips/173) |
| 176 | [Returns over output params](https://abseil.io/tips/176) |
| 197 | [Reader locks should be rare](https://abseil.io/tips/197) |

### Blog Posts

Selected technical deep-dives: [abseil.io/blog](https://abseil.io/blog/)

| Post | Topic |
|------|-------|
| [TCMalloc](https://abseil.io/blog/20200212-tcmalloc) | Memory allocator |
| [Atomic Operations](https://abseil.io/blog/20220118-atomic) | Concurrency dangers |
| [B-tree Containers](https://abseil.io/blog/20190812-btree) | Data structures |
