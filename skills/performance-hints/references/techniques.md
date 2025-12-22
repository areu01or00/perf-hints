# Performance Optimization Techniques

Detailed techniques from Jeff Dean & Sanjay Ghemawat's Performance Hints.

Source: [abseil.io/fast](https://abseil.io/fast/hints.html)

---

## Back-of-Envelope Calculation Examples

### Example: Time to Quicksort 1 Billion 4-Byte Numbers

As a rough approximation, a good quicksort algorithm makes log(N) passes over an array of size N. On each pass, the array contents will be streamed from memory into the processor cache, and the partition code will compare each element once to a pivot element. Let's add up the dominant costs:

1. **Memory bandwidth**: The array occupies 4 GB (4 bytes × 1 billion numbers). Assuming ~16GB/s of memory bandwidth per core, each pass takes ~0.25s. N is ~2³⁰, so we make ~30 passes. **Total memory transfer cost: ~7.5 seconds.**

2. **Branch mispredictions**: We do N×log(N) comparisons, i.e., ~30 billion comparisons. Assume half (15 billion) are mispredicted. At 5 ns per misprediction: **misprediction cost: 75 seconds.** (Correctly predicted branches are essentially free.)

3. **Total estimate: ~82.5 seconds**

**The insight**: Branch mispredictions dominate! Radix sort, which avoids comparisons, would take only ~2.5 seconds (10 memory transfers of 4GB at 16GB/s) instead of 82.5 seconds.

*Refinement*: With a 32MB L3 cache holding 2²³ numbers, the last 22 passes can operate on cache-resident data, cutting memory transfer cost to 2.5 seconds. But this refinement doesn't change the conclusion—branch mispredictions still dominate.

### Example: Time to Generate a Web Page with 30 Image Thumbnails

Original images stored on disk, each approximately 1MB.

**Design 1 - Serial reads**: Each read takes one seek (5ms) + one transfer (10ms) = 15ms per image. For 30 images: **450ms**.

**Design 2 - Parallel on distributed filesystem**: Resource usage is the same, but latency drops by roughly a factor of K disks. With hundreds of disks: **~15ms expected latency**.

**Design 3 - Single SSD**: 20µs + 1ms per image. For 30 images: **~30ms**.

---

## API Considerations

### Organize for Deep Modules

Try to organize code so that performance improvements can be made inside an encapsulation boundary without affecting public interfaces. This will be easier if your modules are deep (significant functionality accessed via a narrow interface).

### Resist Feature Creep

Widely used APIs come under heavy pressure to add features. Be careful when adding new features since these will constrain future implementations and increase cost unnecessarily for users who don't need the new features.

**Example**: Many C++ standard library containers promise iterator stability, which in typical implementations increases the number of allocations significantly, even though many users do not need pointer stability.

### Bulk APIs

Provide bulk operations to reduce expensive API boundary crossings or to take advantage of algorithmic improvements.

**Key insight**: When adding a bulk interface, consider simplifying the signature for the new bulk variant—clients often need less from bulk operations. Example: A bulk lookup that returns bool (were all keys found?) instead of returning a Status object for each key.

Benefits of bulk APIs:
- Amortize locking overhead across multiple operations
- Enable algorithmic improvements (e.g., Floyd's heap construction is O(N) vs O(N log N) for one-at-a-time)
- Reduce function call overhead
- Enable better cache utilization

### View Types

Prefer view types for function arguments (unless ownership is being transferred):
- `std::string_view` instead of `const std::string&`
- `std::Span<T>` instead of `const std::vector<T>&`
- `absl::FunctionRef<R(Args...)>` instead of `std::function`

Benefits:
- Reduces copying
- Allows callers to pick their own container types

### Pre-allocated/Pre-computed Arguments

For frequently called routines, allow higher-level callers to pass in:
- Scratch space they own (avoids allocation in hot path)
- Information the called routine needs that the client already has (avoids recomputation)

Example: Pass an already-computed timestamp to RecordRPC() instead of calling WallTime_Now() again inside the function.

### Thread-Compatible vs Thread-Safe Types

Most generally used types should be **thread-compatible** (synchronized externally). This way callers who do not need thread-safety don't pay for it.

However, if the typical use of a type needs synchronization, prefer to move the synchronization inside the type. This allows the synchronization mechanism to be tweaked as necessary to improve performance (e.g., sharding to reduce contention) without affecting callers.

---

## Algorithmic Improvements

The most critical opportunities for performance improvements come from algorithmic improvements. These opportunities are rare in stable code, but worth paying attention to when writing new code.

### Common Patterns

1. **Replace O(N log N) with O(N) using hash tables**: Sorted intersection can be replaced with hash table lookup.

2. **Add nodes in topological order**: Building a graph incrementally requires expensive work per edge. Adding the entire graph in reverse post-order makes cycle detection trivial.

3. **Replace IntervalMap with hash table**: If adjacent blocks can be found by offset lookup, O(1) hash table beats O(log N) interval map.

4. **Implement good hash functions**: A bad hash function turns O(1) into O(N) due to collisions.

---

## Better Memory Representation

Careful consideration of memory footprint and cache footprint can yield big savings. These techniques focus on supporting common operations by touching fewer cache lines.

### Compact Data Structures

Use compact representations for data that will be accessed often or comprises a large portion of the application's memory usage. A compact representation can significantly reduce memory usage and improve performance by:
- Touching fewer cache lines
- Reducing memory bus bandwidth usage

### Memory Layout

Carefully consider the memory layout of types that have a large memory or cache footprint:

1. **Reorder fields to reduce padding** between fields with different alignment requirements.

2. **Use smaller numeric types** where the stored data will fit.

3. **Enum values sometimes take up a whole word** unless you're careful. Consider using a smaller representation (e.g., `enum class OpType : uint8_t { ... }` instead of `enum class OpType { ... }`).

4. **Order fields so that fields frequently accessed together are closer** to each other—reduces cache lines touched.

5. **Place hot read-only fields away from hot mutable fields** so writes don't cause read-only fields to be evicted from nearby caches.

6. **Move cold data** so it does not live next to hot data—either at the end of the struct, behind a level of indirection, or in a separate array.

### Indices Instead of Pointers

On 64-bit machines, pointers take 64 bits. For pointer-rich data structures:
- Use integer indices into an array instead of pointers
- Indices can be 32 bits or fewer if the count is small enough
- Storage for all elements will be contiguous, leading to better cache locality

### Batched Storage

Avoid data structures that allocate a separate object per stored element (e.g., `std::map`, `std::unordered_map`). Instead use:
- Types with chunked or flat representations (`std::vector`, `absl::flat_hash_map`)
- These have much better cache behavior and less allocator overhead

One useful technique: partition elements into chunks where each chunk holds a fixed number of elements.

### Inlined Storage

Some container types provide space for a small number of elements at the top level, completely avoiding allocations when the number of elements is small. This helps when:
- Instances are constructed often (e.g., stack variables in frequently executed code)
- Many instances are live at the same time

**Caveat**: If `sizeof(T)` is large, inlined storage containers may not be the best choice.

### Nested vs Flat Maps

**Flatten when keys are small**: Replace `btree<a, btree<b, c>>` with `btree<pair<a,b>, c>` to reduce allocations and lookups.

**Keep nested when first key is large and repeated**: If a path occurs in ~1000 keys on average, nested maps reduce memory for storing paths by 1000x and speed up batched accesses.

### Arenas

Arenas help reduce memory allocation cost and have the benefit of:
- Packing independently allocated items next to each other
- Typically in fewer cache lines
- Eliminating most destruction costs

**Caveat**: Easy to misuse by putting too many short-lived objects in a long-lived arena, which bloats memory footprint.

### Arrays Instead of Maps

If the domain of a map can be represented by a small integer or enum, or if the map will have very few elements, the map can be replaced by an array or vector.

### Bit Vectors Instead of Sets

If the domain of a set can be represented by a small integer, the set can be replaced with a bit vector. Set operations become nicely efficient using bitwise boolean operations (OR for union, AND for intersection, etc.).

---

## Reduce Allocations

Memory allocation adds costs:

1. **Time spent in the allocator**
2. **Expensive initialization/destruction** of newly-allocated objects
3. **Cache footprint**: Every allocation tends to be on a new cache line. Data spread across many allocations has larger cache footprint than data in fewer allocations.

### Techniques to Reduce Allocations

1. **Static singleton for empty objects**: Share a static empty instance instead of allocating new empty objects.

2. **Pre-size vectors**: Use `reserve()` or resize before populating to avoid reallocation during push_back.

3. **Static buffers for common cases**: Use a static buffer for typical sizes, allocate only when needed for larger sizes.

4. **Reuse temporary objects**: Keep a persistent buffer/object that gets reused across calls instead of allocating fresh each time.

---

## Avoid Unnecessary Work

### Fast Paths for Common Cases

Provide inexpensive short-circuit paths for common cases to avoid unnecessary work.

Examples:
- Skip locking when there's only one thread
- Return early when input is empty
- Use simpler algorithm for small inputs

### Precompute Expensive Information Once

Move expensive computations out of hot paths by computing once and caching.

### Move Expensive Computations Outside Loops

Pull invariant computations outside of loops rather than recomputing on every iteration.

### Defer Expensive Computation

Defer work until it's actually needed. Sometimes the results are never needed.

### Specialize Code

Replace general-purpose code with specialized versions for hot paths.

Example: Custom integer-to-string printing can be 4x faster than sprintf because it:
- Avoids parsing format string
- Avoids checking for special cases that don't apply
- Can use optimized digit extraction

### Use Caching to Avoid Repeated Work

Cache results of expensive computations for reuse.

**Warning**: Caching adds complexity and memory overhead. Only cache when:
- The computation is expensive enough to justify it
- Results are likely to be reused
- Cache invalidation is manageable

### Make the Compiler's Job Easier

1. **Avoid function calls in hot loops**: Inline small functions or use compiler hints.

2. **Use const where possible**: Helps compiler optimize.

3. **Avoid aliasing**: Use restrict or local variables to help compiler assume no aliasing.

4. **Hand-unroll loops**: For very hot, small loops, manual unrolling can help.

### Reduce Stats Collection Costs

Instrumentation and metrics collection can dominate runtime if not done carefully:
- Batch stats updates
- Use thread-local counters with periodic aggregation
- Sample instead of recording every event

---

## Code Size Considerations

Smaller code can mean better I-cache utilization:
- Avoid excessive inlining of large functions
- Use [[unlikely]] hints to move cold code out of hot paths
- Consider code size when templating

---

## Parallelization and Synchronization

### Reduce False Sharing

Place hot read-only fields away from hot mutable fields. When multiple threads access the same cache line and one writes, all caches with that line must be invalidated.

### Reduce Lock Contention

- Use finer-grained locks
- Use lock-free algorithms where appropriate
- Batch operations under a single lock acquisition

### Use Appropriate Synchronization Primitives

- Atomics for simple counters
- Reader-writer locks when reads dominate
- Consider per-thread data with periodic merging

---

## Summary of Key Techniques

| Category | Technique | Typical Improvement |
|----------|-----------|---------------------|
| Algorithm | Replace O(N log N) with O(N) hash | 20-50%+ |
| Memory | Bit vectors instead of sets | 25-30% |
| Memory | Arrays instead of maps | 10-30% |
| Memory | Flatten nested maps | 10-20% |
| Allocation | Static singleton for empty objects | 20%+ |
| Allocation | Pre-size collections | 10-30% |
| API | Bulk operations | 10-50%+ |
| API | Move instead of copy | 10-15% |
| Cache | Compact data structures | 20-40% |
| Cache | Better memory layout | 10-30% |
