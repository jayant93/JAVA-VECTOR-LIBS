# JScaNN — Migration Plan: Python ScaNN → Java 26

## Overview
ScaNN (Scalable Nearest Neighbors) by Google is a high-performance vector similarity search library optimized for maximum recall at given latency budgets using anisotropic quantization and SIMD.

## Java 26 Equivalent
**JVector** (DiskANN-based) as the Java alternative for scalable ANN.

## Why Java 26
- **Vector API (JEP 489):** SIMD-accelerate quantized distance computation using `ByteVector` and `FloatVector`.
- **Panama FFM:** Direct ScaNN C++ library binding via FFM API for native performance.
- **Virtual Threads:** Concurrent multi-query execution against partitioned index.
- **Structured Concurrency:** Parallel coarse quantizer + fine re-ranking stages.

## GPU / TPU Support
- ScaNN GPU support via CUDA-accelerated distance computation.
- JVector provides competitive performance to ScaNN on CPU with Java Vector API SIMD.
- Panama FFM binding to ScaNN C++ library can unlock GPU capabilities from Java.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Use JVector as the primary Java ANN library — DiskANN offers similar recall/latency tradeoffs.
- For ScaNN-specific needs: bind via Panama FFM to `libscann.so`.
- Replace `searcher.search(query, final_num_neighbors)` with JVector `graph.search(query, k, ef)`.
- Tune `ef_construction` and `beam_width` in JVector to match ScaNN's accuracy.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (ScaNN):**
```python
import scann
searcher = scann.scann_ops_pybind.builder(dataset, 10, 'dot_product')
    .tree(num_leaves=2000, num_leaves_to_search=100)
    .score_ah(2)
    .reorder(100).build()
neighbors, distances = searcher.search(query)
print(neighbors)
```

**Java 26 (JScaNN):**
```java
import io.github.jbellis.jvector.graph.*;
import io.github.jbellis.jvector.vector.*;
// JVector DiskANN — comparable recall/latency to ScaNN
GraphIndexBuilder<float[]> builder = new GraphIndexBuilder<>(
    vectors, VectorEncoding.FLOAT32,
    VectorSimilarityFunction.DOT_PRODUCT, 32, 100, 1.5f, 1.4f);
OnHeapGraphIndex<float[]> index = builder.build();
SearchResult result = GraphSearcher.search(
    query, 10, vectors, VectorSimilarityFunction.DOT_PRODUCT, index, Bits.ALL);
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
