# JHnswlib — Migration Plan: Python Hnswlib → Java 26

## Overview
Hnswlib implements the Hierarchical Navigable Small World graph algorithm for fast approximate nearest neighbor search. It is lightweight, header-only in C++, and widely embedded in production systems.

## Java 26 Equivalent
**Hnswlib Java** (`jelmerk/hnswlib`) and **Hnswlib-JNA** for native bindings.

## Why Java 26
- **Vector API:** SIMD-accelerate distance computation within Java HNSW implementation.
- **Panama FFM:** Zero-JNI binding to hnswlib C++ library for maximum throughput.
- **Virtual Threads:** Concurrent HNSW graph traversal queries.
- **Structured Concurrency:** Parallel ANN queries with result merging.

## GPU / TPU Support
- HNSW graph traversal is CPU-bound; GPU not typically used.
- For GPU ANN: use JVector with CUDA-precomputed distances.
- FAISS GPU index via Panama FFM for billion-scale GPU-accelerated ANN.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Add `com.github.jelmerk/hnswlib-core` Maven dependency.
- Replace `hnswlib.Index('l2', dim)` with `HnswIndex.newBuilder(dim, DistanceFunctions.FLOAT_EUCLIDEAN_DISTANCE, maxElements)`.
- Replace `index.add_items(data)` with `index.addItem(item)`.
- Replace `index.knn_query(query, k)` with `index.findNearest(query, k)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Hnswlib):**
```python
import hnswlib
index = hnswlib.Index(space='l2', dim=128)
index.init_index(max_elements=10000, ef_construction=200, M=16)
index.add_items(data, ids)
labels, distances = index.knn_query(query, k=10)
print(labels)
```

**Java 26 (JHnswlib):**
```java
import com.github.jelmerk.knn.hnsw.*;
import com.github.jelmerk.knn.*;
HnswIndex<String, float[], Item<String, float[]>, Float> index =
    HnswIndex.newBuilder(128, DistanceFunctions.FLOAT_EUCLIDEAN_DISTANCE, 10_000)
        .withEfConstruction(200).withM(16).build();
index.addItem(new Item<>("id1", new float[]{...}));
List<SearchResult<Item<String, float[]>, Float>> results =
    index.findNearest(queryVector, 10);
results.forEach(r -> System.out.println(r.item().id() + " " + r.distance()));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>com.github.jelmerk</groupId>
    <artifactId>hnswlib-core</artifactId>
    <version>1.1.2</version>
</dependency>
```
