# JDGL — Migration Plan: Python DGL (Deep Graph Library) → Java 26

## Overview
DGL (Deep Graph Library) is a framework-agnostic GNN library supporting PyTorch, TF, and MXNet backends. It provides efficient sparse tensor operations for message passing on graphs.

## Java 26 Equivalent
**DL4J + JGraphT** for graph deep learning.

## Why Java 26
- **Virtual Threads:** Parallel message passing across graph partitions.
- **Vector API:** SIMD-accelerate node feature aggregation.
- **Structured Concurrency:** Fan-out neighbor sampling for mini-batch training.
- **Panama FFM:** Bind to native C++ graph processing libraries.

## GPU / TPU Support
GPU-accelerated GNN via DJL PyTorch CUDA for exported GNN models. ND4J CUDA for sparse tensor operations in message passing. Multi-GPU training via DL4J ParallelWrapper.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export GNN models from Python (PyG/DGL) to TorchScript or ONNX.
- Load in DJL for GPU inference.
- Use JGraphT for graph structure and algorithm operations.
- Implement custom message passing with virtual threads for CPU GNN.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (DGL (Deep Graph Library)):**
```python
import dgl
import torch
g = dgl.graph(([0,1,2],[1,2,3]))
conv = dgl.nn.GraphConv(10, 5)
feat = torch.randn(4, 10)
result = conv(g, feat)
```

**Java 26 (JDGL):**
```java
import org.jgrapht.graph.*;
import ai.djl.pytorch.engine.*;
// Export DGL model to TorchScript for DJL inference
Criteria<NDList, NDList> criteria = Criteria.builder()
    .setTypes(NDList.class, NDList.class)
    .optModelPath(Paths.get("dgl_model.pt"))
    .optEngine("PyTorch").build();
ZooModel<NDList, NDList> model = criteria.loadModel();
// Build adjacency tensor from JGraphT graph structure
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.jgrapht</groupId>
    <artifactId>jgrapht-core</artifactId>
    <version>1.5.2</version>
</dependency>
```
