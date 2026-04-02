# JAnnoy — Migration Plan: Python Annoy → Java 26

## Overview
Annoy (Approximate Nearest Neighbors Oh Yeah) by Spotify builds static memory-mapped index files using random projection trees. It is simple, fast at query time, but requires full rebuild on index updates.

## Java 26 Equivalent
**Voyager (Java bindings)** — Spotify's HNSW-based successor to Annoy.

## Why Java 26
- **Panama FFM:** Direct Annoy C++ library binding for mmap-based index access.
- **Virtual Threads:** Concurrent read-only ANN queries against shared mmap index.
- **Vector API:** SIMD-accelerate dot product and L2 distance in tree traversal.
- **Structured Concurrency:** Fan-out queries across multiple index shards.

## GPU / TPU Support
- Annoy index traversal is CPU-bound; GPU not typically beneficial.
- For GPU ANN: migrate to FAISS GPU or JVector with GPU-computed distances.
- Static mmap indexes allow zero-copy shared access across virtual threads.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Prefer Voyager Java over direct Annoy binding — same algorithm, better Java API.
- Replace `annoy.AnnoyIndex(dim, 'angular')` with `Voyager.Index.create(Space.Cosine, dim)`.
- Replace `index.build(50)` with Voyager `index` (auto-built).
- Replace `index.get_nns_by_vector(query, k)` with `index.query(query, k)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Annoy):**
```python
from annoy import AnnoyIndex
index = AnnoyIndex(128, 'angular')
for i, v in enumerate(vectors):
    index.add_item(i, v)
index.build(50)
nns = index.get_nns_by_vector(query, 10)
print(nns)
```

**Java 26 (JAnnoy):**
```java
import com.spotify.voyager.jni.*;
import com.spotify.voyager.jni.Index.*;
Index index = new Index(SpaceType.Cosine, 128);
for (int i = 0; i < vectors.length; i++)
    index.addItem(vectors[i], i);
// Query
QueryResults results = index.query(queryVector, 10);
System.out.println(Arrays.toString(results.getLabels()));
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
