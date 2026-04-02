# JCuPy — Migration Plan: Python CuPy → Java 26

## Overview
CuPy is a NumPy-compatible array library that executes operations directly on NVIDIA GPUs via CUDA, enabling massive speedups for numerical computation without changing the NumPy-style API.

## Java 26 Equivalent
**ND4J CUDA backend** for GPU-accelerated n-dimensional array operations.

## Why Java 26
- **Panama FFM:** Direct CUDA runtime calls via foreign function bindings, bypassing JNI overhead.
- **Vector API:** CPU fallback path with SIMD for host-side pre/post processing.
- **Virtual Threads:** Asynchronous GPU kernel launches without blocking carrier threads.
- **Structured Concurrency:** Coordinate multi-GPU kernel submissions safely.

## GPU / TPU Support
- ND4J CUDA backend maps all array ops to CUDA kernels automatically.
- Switch backend with a single Maven dependency swap: `nd4j-cuda-12.x`.
- Multi-GPU support via `CudaEnvironment.getInstance().getConfiguration().allowMultiGPU(true)`.
- TPU routing available via TF Java API on Cloud TPU nodes.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `cp.array(...)` with `Nd4j.create(...)` on the CUDA backend.
- Replace CuPy ufuncs with ND4J element-wise ops — same result, same device.
- Use `Nd4j.getAffinityManager().replicateAsArray()` for multi-GPU data distribution.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (CuPy):**
```python
import cupy as cp
a = cp.array([1.0, 2.0, 3.0])
b = cp.array([4.0, 5.0, 6.0])
c = cp.dot(a, b)
print(float(c))
```

**Java 26 (JCuPy):**
```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;
// CUDA backend active via nd4j-cuda-12.x dependency
INDArray a = Nd4j.create(new float[]{1,2,3});
INDArray b = Nd4j.create(new float[]{4,5,6});
float c = a.mmul(b.transpose()).getFloat(0);
System.out.println(c);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.3-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```
