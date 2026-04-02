# JMXNet — Migration Plan: Python MXNet → Java 26

## Overview
Apache MXNet is a scalable deep learning framework supporting imperative and symbolic programming, Gluon high-level API, and efficient training across CPUs, GPUs, and distributed clusters.

## Java 26 Equivalent
**DJL MXNet engine** (Apache MXNet Java bindings via DJL).

## Why Java 26
- **Panama FFM:** MXNet C API bindings for low-overhead tensor operations.
- **Virtual Threads:** Asynchronous MXNet operator execution scheduling.
- **Vector API:** CPU fallback SIMD for host-side data preprocessing.
- **Structured Concurrency:** Multi-context (CPU+GPU) computation coordination.

## GPU / TPU Support
- DJL MXNet engine routes all operations to CUDA-enabled MXNet native lib.
- Multi-GPU training via MXNet's KVStore parameter server.
- Supports FP16 mixed precision training for faster GPU throughput.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Use DJL with MXNet engine: set `Engine.getEngine('MXNet')`.
- Load MXNet `.params` and `.json` model files via DJL `Criteria`.
- Replace Gluon blocks with DJL `SequentialBlock` API.
- Replace `mx.io.DataIter` with DJL `RandomAccessDataset`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (MXNet):**
```python
import mxnet as mx
net = mx.gluon.nn.Sequential()
net.add(mx.gluon.nn.Dense(128, activation='relu'))
net.add(mx.gluon.nn.Dense(10))
net.initialize()
output = net(mx.nd.array(x))
```

**Java 26 (JMXNet):**
```java
import ai.djl.mxnet.engine.MxEngine;
import ai.djl.*;
Criteria<NDList, NDList> criteria = Criteria.builder()
    .setTypes(NDList.class, NDList.class)
    .optModelPath(Paths.get("model"))
    .optEngine("MXNet")
    .build();
ZooModel<NDList, NDList> model = criteria.loadModel();
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ai.djl.mxnet</groupId>
    <artifactId>mxnet-engine</artifactId>
    <version>0.28.0</version>
</dependency>
```
