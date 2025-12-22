# Code Examples from Jeff Dean & Sanjay Ghemawat's Performance Hints

Source: [abseil.io/fast](https://abseil.io/fast/hints.htmlhints.html)

Real before/after examples from Google's internal codebase with actual benchmark results.

---

## Algorithmic Improvements

### Replace O(N log N) with O(N) using hash tables

Old code detected shared sources using sorted intersection. New code uses hash table lookup:

```
name             old time/op  new time/op  delta
BM_CompileLarge   28.5s ± 2%   22.4s ± 2%  -21.61%
```

### Deadlock detection algorithm replacement

Replaced deadlock detection with Pearce-Kelly dynamic topological sort:
- Old: O(|V|²) space, 22 microseconds per InsertEdge at 2K nodes
- New: O(|V|+|E|) space, 0.5 microseconds per InsertEdge, scales to millions of mutexes

```
Old (DEBUG mode):
BM_StressTest/2k        23553 ns
BM_StressTest/4k        45879 ns
BM_StressTest/16k      776938 ns  (hitting performance cliff)

New (DEBUG mode):
BM_StressTest/2k          392 ns   (60x faster)
BM_StressTest/4k          392 ns
BM_StressTest/32k         407 ns
BM_StressTest/256k        456 ns
BM_StressTest/1M          534 ns   (scales linearly)
```

### Replace IntervalMap with hash table

Initial code used IntervalMap (O(log N) lookups) for block coalescing. Hash table suffices since adjacent blocks can be found by offset lookup:

```cpp
// Old: O(log N) lookups
gtl::IntervalMap<int64, BlockState> block_list_;

// New: O(1) lookups with custom hash
struct OffsetHash {
  size_t operator()(int64 value) const {
    uint64 m = value;
    m *= uint64_t{0x9ddfea08eb382d69};
    return static_cast<uint64_t>(m ^ (m >> 32));
  }
};
using BlockTable = absl::flat_hash_map<int64, HashTableEntry, OffsetHash>;
```

Result: **4X performance improvement** in BestFitAllocator.

---

## Bulk API Examples

### Bulk MemoryManager::LookupMany

```cpp
// Old: Per-key lookup returns StatusOr
util::StatusOr<LiveTensor> Lookup(const TensorIdProto& id);

// New: Bulk lookup simplifies signature (clients only need bool)
struct LookupKey {
  ClientHandle client;
  uint64 local_id;
};
bool LookupMany(absl::Span<const LookupKey> keys,
                absl::Span<tensorflow::Tensor> tensors);
```

### Bulk ObjectStore::DeleteRefs

```cpp
// Old: Lock per delete
absl::Status DeleteRef(Ref);

// New: Single lock for batch
absl::Status DeleteRefs(absl::Span<const Ref> refs) {
  util::Status result;
  absl::MutexLock l(&mu_);  // One lock for entire batch
  for (auto ref : refs) {
    result.Update(DeleteRefLocked(ref));
  }
  return result;
}
```

### Cache decoded results for future calls

```cpp
// Old: Decode all entries up to pos on every call
void GetTokenString(int pos, std::string* out) const {
  absl::FixedArray<LexiconEntry, 32> entries(pos + 1);
  for (int i = 0; i <= pos; ++i) {
    // decode entry i...
  }
}

// New: Cache decoded blocks, return from cache on hit
mutable std::vector<absl::InlinedVector<std::string, 16>> cache_;
void GetTokenString(int pos, std::string* out) const {
  if (!cache_[skentry].empty()) {
    *out = cache_[skentry][pos];
    return;
  }
  // decode and populate cache...
}
```

---

## Reduce Allocations Examples

### Static singleton for empty objects (21% improvement)

```cpp
// Old: Allocates new DeviceInfo every time dinfo is null
LiveTensor::LiveTensor(tf::Tensor t, std::shared_ptr<const DeviceInfo> dinfo)
    : tensor(std::move(t)),
      device_info(dinfo ? std::move(dinfo) : std::make_shared<DeviceInfo>()) {}

// New: Share static empty instance
static const std::shared_ptr<DeviceInfo>& empty_device_info() {
  static std::shared_ptr<DeviceInfo>* result =
      new std::shared_ptr<DeviceInfo>(new DeviceInfo);
  return *result;
}

LiveTensor::LiveTensor(tf::Tensor t, std::shared_ptr<const DeviceInfo> dinfo) {
  if (dinfo) {
    device_info = std::move(dinfo);
  } else {
    device_info = empty_device_info();  // No allocation
  }
}
```

### Pre-size vector instead of push_back

```cpp
// Old: N push_back operations, potential reallocs
for (int i = 0; i < ndocs-1; i++) {
  docs_.push_back(DocId(...));
}

// New: Pre-size, direct write
docs_.resize(ndocs);
DocId* docptr = &docs_[0];
for (int i = 0; i < ndocs-1; i++) {
  *docptr = DocId(...);
  docptr++;
}
```

### Static zero buffer for common case

```cpp
// Old: Allocate and memset every call
std::unique_ptr<quint8[]> zero_data(new quint8[max_embedding_width]);
memset(zero_data.get(), 0, sizeof(quint8) * max_embedding_width);

// New: Static buffer for typical case, allocate only when needed
static const int kTypicalMaxEmbedding = 256;
static quint8 static_zero_data[kTypicalMaxEmbedding];  // All zeroes

quint8* zero_data;
if (max_embedding_width <= ARRAYSIZE(static_zero_data)) {
  zero_data = &static_zero_data[0];  // No allocation
} else {
  zero_data_backing = make_unique<quint8[]>(max_embedding_width);
  memset(zero_data_backing.get(), 0, ...);
  zero_data = zero_data_backing.get();
}
```

---

## Bit Vector Examples

### Spanner ZoneSet (29% improvement)

```cpp
// Old: Hash set
class ZoneSet: public dense_hash_set<ZoneId> {
  bool Contains(ZoneId zone) const {
    return count(zone) > 0;
  }
};

// New: Bit vector
class ZoneSet {
  bool ContainsZone(ZoneId zone) const {
    return zone < b_.size() && b_.get_bit(zone);
  }
 private:
  int size_;
  util::bitmap::InlinedBitVector<256> b_;
};
```

```
Benchmark                Base (ns)  New (ns)  Improvement
BM_Evaluate/1                  960       676    +29.6%
BM_Evaluate/2                 1661      1138    +31.5%
BM_Evaluate/10                7819      5739    +26.6%
BM_Evaluate/40               36836     26430    +28.2%
```

### HLO Reachability Map

```cpp
// Old: Hash map of hash sets
using TransitiveOperandMap =
    std::unordered_map<const HloInstruction*,
                       std::unordered_set<const HloInstruction*>>;

// New: Bit matrix
class HloComputation::ReachabilityMap {
  // Dense id assignment from HloInstruction* to number
  tensorflow::gtl::FlatMap<const HloInstruction*, int> ids_;
  // matrix_(a,b) is true iff b is reachable from a
  tensorflow::core::Bitmap matrix_;
};
```

---

## Thread Safety: Remove Unnecessary Locking

```cpp
// Old: Lock on simple getter when callers already synchronized
TransferPhase HitlessTransferPhase::get() const {
  static CallsiteMetrics cm("HitlessTransferPhase::get");
  MonitoredMutexLock l(&cm, &mutex_);
  return phase_;
}

// New: Direct return (callers handle synchronization)
TransferPhase HitlessTransferPhase::get() const { return phase_; }
```

---

## Nested Maps: When to Flatten vs Keep Nested

### Flatten when keys are small

```cpp
// Old: Two lookups
absl::btree_map<std::string, absl::btree_map<std::string, OpDef>> ops;

// New: Single lookup with compound key
absl::btree_map<std::pair<absl::string_view, absl::string_view>,
                const OpDef*> ops;
```

### Keep nested when first key is large and repeated

When path occurs in ~1000 keys on average, nested maps reduced memory for storing paths by 1000x and sped up batched accesses:

**76% performance improvement** by switching TO nested maps in this case.

---

## Move Instead of Copy

### Tensor via gRPC (10-15% improvement)

```
Old: BM_RPC/30/98k_mean    148764691 ns
New: BM_RPC/30/98k_mean    131595940 ns
```

### Large options structure

```cpp
// Old: Copy
return DocPLIteratorFactory::Create(opts);

// New: Move
return DocPLIteratorFactory::Create(std::move(opts));
```

### std::sort vs std::stable_sort

```cpp
// Old: stable_sort does internal copy
std::stable_sort(hits_.begin(), hits_.end(), ...);

// New: Define operator< for stable ordering, use sort
struct HitWithPayloadOffset {
  bool operator<(const HitWithPayloadOffset& other) const {
    return (docid < other.docid) ||
           (docid == other.docid &&
            first_payload_offset < other.first_payload_offset);
  }
};
std::sort(hits_.begin(), hits_.end());
```

---

## Array Instead of Map

```cpp
// Old: flat_map for payload type → clock frequency
const gtl::flat_map<int, int> payload_type_to_clock_frequency_;

// New: Direct array indexing (payload_type is small int)
struct PayloadTypeToClockRateMap {
  int map[128];
};
const PayloadTypeToClockRateMap payload_type_to_clock_frequency_;
```

---

## Pre-allocated/Pre-computed Arguments

```cpp
// Old: RecordRPC fetches current time internally
static void RecordRPC(const Name &name, const RPC_Stats_Measurement& m);

// New: Caller passes already-computed time
static void RecordRPC(const Name &name, const RPC_Stats_Measurement& m,
                      WallTime now);

// Caller already has 'now' from earlier:
const WallTime now = WallTime_Now();
// ... other work using 'now' ...
RPC_Stats::RecordRPC(stats_name, m, now);  // No redundant time fetch
```

---

## Graph Construction: Bulk Initialization

```cpp
// Old: Add nodes and edges one at a time (expensive per-edge checks)
graph.PerNode([&](Node node) {
  graph_->AddNode(node);
});
PerEdge(graph, [&](Edge e) {
  if (!graph_->InsertEdge(e.src, e.dst)) {
    // cycle detected
  }
});

// New: Initialize from existing graph in one shot
// Uses DFS to assign ranks in topological order
// Cycle detection becomes a simple rank comparison
bool GraphCycles::InitFrom(const util_graph::Graph& src) {
  // Assign ranks in reverse post-order via DFS
  auto leave = [&](Node node) {
    r->rank[node] = --last_rank;
  };
  DFSAll(src, std::nullopt, leave);

  // Add edges with O(1) cycle check
  PerEdge(src, [&](Edge e) {
    if (r->rank[e.src] >= r->rank[e.dst]) {
      have_cycle = true;
    }
  });
}
```

---

## Good Hash Function Implementation

```cpp
// Old: Simple hash on address only
struct LocationHash {
  size_t operator()(const Location* key) const {
    return key != nullptr ? util_hash::Hash(key->address()) : 0;
  }
};

// New: Hash combining all relevant fields
size_t HashLocation(const Location& loc) {
  util_hash::MurmurCat m;

  // Encode simple boolean features into a single value
  m.AppendAligned((loc.dynamic() ? 1 : 0)
                  | (loc.append_shard_to_address() ? 2 : 0)
                  | (loc.is_any() ? 4 : 0)
                  | (!loc.any_of().empty() ? 8 : 0)
                  | (loc.has_shardmap() ? 16 : 0)
                  | (loc.has_sharding() ? 32 : 0));

  if (loc.has_shardmap()) {
    m.AppendAligned(loc.shardmap().output() |
                    static_cast<uint64_t>(loc.shardmap().stmt()) << 20);
  }
  // ... handle other fields ...

  add_string(loc.address());
  add_string(loc.lb_policy());

  return m.GetHash();
}
```

---

## Source

All examples from: Jeff Dean and Sanjay Ghemawat, "Performance Hints"
https://abseil.io/fast/hints.html
