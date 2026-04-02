# JRay — Migration Plan: Python Ray → Java 26

## Overview
Ray is a distributed computing framework for ML — training, hyperparameter tuning, and serving. It provides actor-based parallelism and a task graph scheduler.

## Java 26 Equivalent
**Apache Flink** and **Apache Storm** for distributed ML workloads.

## Why Java 26
- **Virtual Threads:** Replace Python async task model with JVM virtual threads.
- **Structured Concurrency:** Safe parallel computation with shutdown-on-failure.
- **Vector API:** SIMD-accelerate partition-level numerical operations.
- **Panama FFM:** Zero-copy data sharing between JVM partitions.

## GPU / TPU Support
NVIDIA RAPIDS GPU acceleration for Spark (GPU DataFrames). Flink GPU support via Docker containers with CUDA. GPU preprocessing pipelines via DJL feed into distributed computation.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Set up Spark/Flink cluster or local mode.
- Replace Python DataFrame ops with Java Dataset/DataStream API.
- Replace `@ray.remote` with `StructuredTaskScope.fork()`.
- Configure GPU resources in cluster YARN/K8s configuration.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Ray):**
```python
import ray
ray.init()
@ray.remote
def train_model(shard): return model.fit(shard)
futures = [train_model.remote(shard) for shard in shards]
results = ray.get(futures)
```

**Java 26 (JRay):**
```java
import java.util.concurrent.*;
// Virtual threads as Ray remote equivalents
try (var scope = StructuredTaskScope.ShutdownOnFailure()) {
    List<StructuredTaskScope.Subtask<Model>> tasks = shards.stream()
        .map(shard -> scope.fork(() -> trainModel(shard)))
        .toList();
    scope.join().throwIfFailed();
    List<Model> models = tasks.stream().map(StructuredTaskScope.Subtask::get).toList();
}
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<!-- Use Java 26 built-in structured concurrency; no external dependency needed -->
<!-- For large cluster: use Apache Flink or Spark -->
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-java</artifactId>
    <version>1.20.0</version>
</dependency>
```
