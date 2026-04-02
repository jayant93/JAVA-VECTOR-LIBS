# JVoyager — Migration Plan: Python Voyager → Java 26

## Overview
Voyager is Spotify's production-ready nearest-neighbor search library using the HNSW algorithm. It provides official Java and Python bindings, is thread-safe, and supports serialization for production deployment.

## Java 26 Equivalent
**Voyager (official Java bindings)** by Spotify.

## Why Java 26
- **Virtual Threads:** Highly concurrent read queries — Voyager indexes are thread-safe.
- **Vector API:** SIMD-accelerate inner product and L2 distance in HNSW traversal.
- **Panama FFM:** Direct binding to Voyager C++ core for zero-JNI overhead.
- **Structured Concurrency:** Fan-out batch queries with timeout and cancellation.

## GPU / TPU Support
- HNSW traversal is CPU-bound; Voyager runs efficiently on modern multi-core CPUs.
- For GPU-scale ANN: combine Voyager for small indexes with JVector for large-scale.
- Virtual threads provide near-GPU-level concurrency for query throughput on CPU.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Add Voyager Maven dependency.
- Replace Python `Index(Space.Cosine, dim)` with Java `new Index(SpaceType.Cosine, dim)`.
- Replace `index.add_items(vectors, ids)` with `index.addItems(vectors, ids)`.
- Replace `index.query(v, k)` with `index.query(v, k)` — identical API.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Voyager):**
```python
from voyager import Index, Space
index = Index(Space.Cosine, num_dimensions=128)
index.add_items(vectors, ids)
neighbors, distances = index.query(query_vector, k=10)
print(neighbors, distances)
```

**Java 26 (JVoyager):**
```java
import com.spotify.voyager.jni.*;
import com.spotify.voyager.jni.Index.*;
Index index = new Index(SpaceType.Cosine, 128);
index.addItems(vectorMatrix, ids, -1, 4); // 4 threads
QueryResults result = index.query(queryVector, 10);
System.out.println(Arrays.toString(result.getLabels()));
System.out.println(Arrays.toString(result.getDistances()));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>com.spotify</groupId>
    <artifactId>voyager</artifactId>
    <version>2.0.9</version>
</dependency>
```
