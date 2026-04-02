# JNumPy — Migration Plan: Python NumPy → Java 26

## Overview
NumPy is the foundational numerical computing library for Python, providing n-dimensional
arrays, broadcasting, linear algebra, FFT, and random number generation. It underpins
nearly every scientific Python library and is the de-facto standard for array manipulation.

## Java 26 Equivalent
**ND4J** (N-Dimensional Arrays for Java) from the Eclipse DeepLearning4J ecosystem.
ND4J mirrors NumPy's API closely, supports CPU and GPU backends, and integrates with DL4J.

## Why Java 26
- **Vector API (JEP 489, jdk.incubator.vector):** SIMD-accelerated element-wise array ops
  (add, multiply, dot product) that map directly to CPU vector registers (AVX-512, NEON).
- **Foreign Function & Memory API (JEP 454):** Zero-copy interop with native BLAS/LAPACK
  libraries (OpenBLAS, MKL) via off-heap `MemorySegment`, matching NumPy's C-extension speed.
- **Virtual Threads (Project Loom):** Parallelize independent array batch operations across
  thousands of lightweight threads without OS thread overhead.
- **Structured Concurrency (JEP 480):** Fan-out large matrix operations over sub-tasks and
  collect results safely.

## GPU / TPU Support
- ND4J ships a `nd4j-cuda-12.x` backend; switch backends by changing one Maven dependency.
- CUDA kernels for element-wise ops, GEMM, and reductions run on-device with no code change.
- OpenCL backend (`nd4j-native` + SaaS GPU) available for AMD/Intel GPUs.
- For TPU access, route through TensorFlow Java API on Google Cloud TPU nodes.

## Migration Phases

### Phase 1 – Setup
- Add ND4J + BLAS native dependency to Maven/Gradle.
- Configure backend via `Nd4j.setBackend()` or system property.
- Establish logging and memory workspace configuration.

### Phase 2 – Core Migration
- Replace `np.array(...)` with `Nd4j.create(float[], int[])`.
- Replace NumPy ufuncs (`np.add`, `np.multiply`) with ND4J ops (`array.add()`, `.mul()`).
- Port `np.dot` / `np.matmul` to `Nd4j.gemm()` or `mmul()`.
- Replace `np.random.*` with `Nd4j.getRandom()` equivalents.

### Phase 3 – Optimization
- Use ND4J `Workspace` API for arena-style memory management to reduce GC pressure.
- Enable `Nd4j.getMemoryManager().setAutoGcWindow(0)` to prevent mid-batch GC.
- Profile with async prefetch using virtual threads for data pipeline overlap.

### Phase 4 – GPU/Vector Acceleration
- Switch to `nd4j-cuda-12.x` backend for GPU GEMM and conv operations.
- Use Java Vector API for custom element-wise kernels on CPU fallback paths.
- Leverage `MemorySegment` for pinned GPU memory transfers via Panama FFM.

## API Comparison

**Python (NumPy):**
```python
import numpy as np
a = np.array([[1.0, 2.0], [3.0, 4.0]])
b = np.array([[5.0, 6.0], [7.0, 8.0]])
c = np.dot(a, b)
print(c)
```

**Java 26 (ND4J):**
```java
import org.nd4j.linalg.factory.Nd4j;
import org.nd4j.linalg.api.ndarray.INDArray;

INDArray a = Nd4j.create(new float[][]{{1,2},{3,4}});
INDArray b = Nd4j.create(new float[][]{{5,6},{7,8}});
INDArray c = a.mmul(b);
System.out.println(c);
```

## Performance Considerations
- ND4J delegates BLAS calls to OpenBLAS/MKL via JNI — on-par with NumPy for large matrices.
- Java Vector API (`FloatVector`, `DoubleVector`) gives SIMD throughput for custom reductions.
- Off-heap `INDArray` storage avoids GC pauses during large array operations.
- Virtual threads enable pipelined batch processing without blocking the carrier thread pool.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-native-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
<!-- GPU backend (swap with above for CUDA) -->
<dependency>
    <groupId>org.nd4j</groupId>
    <artifactId>nd4j-cuda-12.3-platform</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```
