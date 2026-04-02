# JFAISS — Migration Plan: Python FAISS → Java 26

## Overview
FAISS (Facebook AI Similarity Search) is Meta's library for billion-scale efficient similarity search and clustering of dense vectors. It supports IVF, HNSW, PQ, and GPU indexes.

## Java 26 Equivalent
**JVector** and **Apache Lucene** dense vector search.

## Why Java 26
- **Vector API (JEP 489):** SIMD-accelerate distance computation (L2, cosine, inner product) using `FloatVector` with AVX-512.
- **Panama FFM:** Bind directly to FAISS C++ library for GPU index access.
- **Virtual Threads:** Concurrent query batches against the vector index.
- **Structured Concurrency:** Fan-out multi-shard search, merge top-k results.

## GPU / TPU Support
- JVector's DiskANN algorithm outperforms HNSW for large-scale GPU-adjacent workloads.
- FAISS GPU index accessible via Panama FFM binding to `faiss_gpu` C++ library.
- Apache Lucene HNSW index uses Java Vector API for SIMD-accelerated distance computation.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `faiss.IndexFlatL2(dim)` with `JVector GraphIndexBuilder` or `Lucene HnswGraphBuilder`.
- Replace `index.add(vectors)` with JVector `builder.add()` or Lucene `Field(type, vector)`.
- Replace `index.search(query, k)` with JVector `graph.search(query, k)` or Lucene `KnnVectorQuery`.
- For IVF-style partitioning: use Lucene's IVF index or JVector's partition-based approach.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (FAISS):**
```python
import faiss
import numpy as np
dim = 128
index = faiss.IndexFlatL2(dim)
vectors = np.random.rand(10000, dim).astype('float32')
index.add(vectors)
D, I = index.search(vectors[:5], k=10)
print(I)
```

**Java 26 (JFAISS):**
```java
import io.github.jbellis.jvector.graph.*;
import io.github.jbellis.jvector.vector.*;
int dim = 128; int k = 10;
VectorSimilarityFunction sim = VectorSimilarityFunction.EUCLIDEAN;
GraphIndexBuilder<float[]> builder =
    new GraphIndexBuilder<>(vectors, VectorEncoding.FLOAT32, sim, 16, 100, 1.5f, 1.4f);
OnHeapGraphIndex<float[]> index = builder.build();
float[] query = vectors.get(0);
SearchResult result = GraphSearcher.search(query, k, vectors, sim, index, Bits.ALL);
System.out.println(Arrays.toString(result.getNodes()));
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
