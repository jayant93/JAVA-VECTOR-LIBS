# JSciPy — Migration Plan: Python SciPy → Java 26

## Overview
SciPy builds on NumPy to provide algorithms for integration, optimization, interpolation,
signal processing, linear algebra, statistics, and sparse matrices. It is the backbone of
scientific computing in Python, used in physics simulations, engineering, and ML research.

## Java 26 Equivalent
**Apache Commons Math** for statistics, linear algebra, optimization, and integration.
**ojAlgo (Ojalgo)** for high-performance linear algebra and mathematical programming with
modern Java idioms and optional BLAS acceleration.

## Why Java 26
- **Vector API (JEP 489):** SIMD-accelerated dense matrix-vector products in Commons Math
  and ojAlgo inner loops; custom signal-processing kernels benefit from `FloatVector` lanes.
- **Foreign Function & Memory API (JEP 454):** Call native LAPACK (MKL/OpenBLAS) routines
  for eigendecomposition and SVD without JNI boilerplate using Panama `Linker`.
- **Virtual Threads:** Parallelize independent optimization runs (e.g., Monte Carlo
  integration sub-intervals) on virtual threads with minimal overhead.
- **Structured Concurrency:** Coordinate multi-start optimization algorithms safely.

## GPU / TPU Support
- For GPU-accelerated linear algebra, bridge matrices to ND4J CUDA backend (SVD, eig).
- Sparse matrix GPU operations via NVIDIA cuSPARSE accessible through JNI/Panama FFM.
- TPU-based linear algebra via TensorFlow Java `tf.linalg` ops on Cloud TPU nodes.

## Migration Phases

### Phase 1 – Setup
- Add Apache Commons Math 3.x + ojAlgo to Maven.
- Identify SciPy sub-module usage (linalg, optimize, signal, stats, integrate).
- Map SciPy function signatures to Commons Math equivalents.

### Phase 2 – Core Migration
- Replace `scipy.linalg.solve(A, b)` with Commons Math `LUDecomposition.getSolver().solve()`.
- Replace `scipy.optimize.minimize(...)` with `SimplexOptimizer` or `CMAESOptimizer`.
- Replace `scipy.integrate.quad(f, a, b)` with `GaussIntegrator` or `RombergIntegrator`.
- Replace `scipy.stats.norm.pdf(x)` with `NormalDistribution.density(x)`.

### Phase 3 – Optimization
- Use ojAlgo `PrimitiveDenseStore` for cache-friendly matrix storage layouts.
- Enable BLAS native acceleration in ojAlgo via `SparseStore` + native delegates.
- Batch independent optimizations using virtual thread executor pools.

### Phase 4 – GPU/Vector Acceleration
- Use Java Vector API for custom FFT butterfly stages and FIR filter convolutions.
- Bridge dense matrices to ND4J for GPU eigendecomposition on large covariance matrices.
- Use Panama FFM to call cuFFT for GPU-accelerated signal processing.

## API Comparison

**Python (SciPy):**
```python
from scipy.linalg import solve
import numpy as np
A = np.array([[3,1],[1,2]], dtype=float)
b = np.array([9,8], dtype=float)
x = solve(A, b)
```

**Java 26 (Apache Commons Math):**
```java
import org.apache.commons.math3.linear.*;

RealMatrix A = new Array2DRowRealMatrix(new double[][]{{3,1},{1,2}});
RealVector b = new ArrayRealVector(new double[]{9,8});
RealVector x = new LUDecomposition(A).getSolver().solve(b);
System.out.println(x);
```

## Performance Considerations
- Commons Math is pure Java; for LAPACK-level speed, ojAlgo with native BLAS is preferred.
- Java Vector API provides 2–4x speedup for custom signal processing filter kernels.
- Virtual threads allow massively parallel Monte Carlo integration without thread exhaustion.
- Panama FFM eliminates JNI overhead for repeated BLAS/LAPACK callsites.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-math3</artifactId>
    <version>3.6.1</version>
</dependency>
<dependency>
    <groupId>org.ojalgo</groupId>
    <artifactId>ojalgo</artifactId>
    <version>53.3.0</version>
</dependency>
```
