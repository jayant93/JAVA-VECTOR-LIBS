# JChroma — Migration Plan: Python Chroma → Java 26

## Overview
Chroma is an open-source embedded vector database for LLM applications. It runs in-process, handles embedding generation via model integration, and provides simple CRUD + similarity search.

## Java 26 Equivalent
**JVector** (embedded) for in-process vector storage and search.

## Why Java 26
- **Virtual Threads:** Concurrent collection operations without blocking.
- **Vector API:** SIMD distance computation inside JVector.
- **Panama FFM:** Off-heap memory for large vector collections.
- **Structured Concurrency:** Parallel add + query operations.

## GPU / TPU Support
- JVector's in-process model runs entirely on CPU with SIMD acceleration.
- GPU-accelerated embedding via DJL/ONNX Runtime GPU, then insert into JVector.
- Future: ND4J CUDA backend for GPU distance computation in JVector.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `chromadb.Client()` with JVector `GraphIndexBuilder` for in-process use.
- Replace `collection.add(embeddings, ids)` with JVector `builder.add(id, vector)`.
- Replace `collection.query(query_embeddings, n_results)` with JVector `GraphSearcher.search()`.
- For persistence: serialize JVector index to disk via `OnDiskGraphIndex`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Chroma):**
```python
import chromadb
client = chromadb.Client()
coll = client.create_collection('docs')
coll.add(embeddings=[[0.1]*128], ids=['doc1'])
results = coll.query(query_embeddings=[[0.1]*128], n_results=5)
print(results['ids'])
```

**Java 26 (JChroma):**
```java
import io.github.jbellis.jvector.graph.*;
import io.github.jbellis.jvector.vector.*;
// In-process vector store — equivalent to Chroma embedded mode
GraphIndexBuilder<float[]> builder = new GraphIndexBuilder<>(
    new ListRandomAccessVectorValues(vectors, dim),
    VectorEncoding.FLOAT32, VectorSimilarityFunction.COSINE, 16, 100, 1.5f, 1.4f);
OnHeapGraphIndex<float[]> index = builder.build();
SearchResult result = GraphSearcher.search(
    queryVec, 5, vectorValues, VectorSimilarityFunction.COSINE, index, Bits.ALL);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>io.github.jbellis</groupId>
    <artifactId>jvector</artifactId>
    <version>3.0.4</version>
</dependency>
```
