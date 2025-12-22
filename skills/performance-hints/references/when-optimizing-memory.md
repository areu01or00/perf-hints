# When Optimizing Memory

Consult these Abseil sources for memory optimization, allocations, cache efficiency, and data structure choices.

## Core Articles

- [Reducing memory bandwidth](https://abseil.io/fast/62) - Memory access patterns
- [Reducing memory indirections](https://abseil.io/fast/83) - Pointer chasing and cache locality

## Data Structures

- [B-tree Containers](https://abseil.io/blog/20190812-btree) - When to use B-trees over hash tables
- [Fixing things with hashtable profiling](https://abseil.io/fast/26) - Hash table optimization

## Memory Allocators

- [TCMalloc](https://abseil.io/blog/20200212-tcmalloc) - Google's high-performance memory allocator

## C++ Tips (Universal Principles)

- [emplace vs push_back](https://abseil.io/tips/112) - Avoiding unnecessary copies
- [Temporaries, moves, copies](https://abseil.io/tips/77) - Understanding object lifecycle
- [Copy elision](https://abseil.io/tips/117) - Compiler optimizations for copies
