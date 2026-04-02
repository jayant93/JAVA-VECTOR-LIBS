# JNNI — Migration Plan: Python NNI (Neural Network Intelligence) → Java 26

## Overview
NNI is Microsoft's AutoML toolkit for neural architecture search, hyperparameter optimization, model pruning, and quantization. It provides state-of-the-art compression algorithms.

## Java 26 Equivalent
**ONNX Runtime Java** with NNI-compressed models.

## Why Java 26
- **Virtual Threads:** Async calibration data feeding for static quantization.
- **Vector API:** SIMD INT8 dot product computation in custom kernels.
- **Panama FFM:** Direct TensorRT engine binding for GPU INT8 inference.
- **Structured Concurrency:** Parallel layer-wise pruning and calibration.

## GPU / TPU Support
ONNX Runtime TensorRT provider enables INT8/FP16 GPU inference. TensorFlow Lite GPU delegate for mobile GPU acceleration. PyTorch INT8 models converted to TensorRT for maximum GPU throughput via ONNX Runtime.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Compress models offline in Python (quantize/prune).
- Export to ONNX for Java deployment.
- Load INT8 ONNX model in Java via ONNX Runtime with GPU/TensorRT provider.
- Benchmark with JMH to verify speedup.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (NNI (Neural Network Intelligence)):**
```python
import nni
from nni.compression.pruning import L1NormPruner
pruner = L1NormPruner(model, [{'sparsity':0.5,'op_types':['Linear']}])
compressed_model, masks = pruner.compress()
pruner.export_model('pruned_model.pth', 'mask.pth')
```

**Java 26 (JNNI):**
```java
import ai.onnxruntime.*;
// Compress model with NNI in Python, export to ONNX, load in Java
// NNI-pruned + ONNX-quantized model
OrtEnvironment env = OrtEnvironment.getEnvironment();
OrtSession.SessionOptions opts = new OrtSession.SessionOptions();
opts.setOptimizationLevel(OrtSession.SessionOptions.OptLevel.ALL_OPT);
opts.addCUDA(0);
try (OrtSession session = env.createSession("nni_compressed.onnx", opts)) {
    OnnxTensor input = OnnxTensor.createTensor(env, inputData);
    System.out.println(session.run(Map.of("input", input)).get(0).getValue());
}
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>com.microsoft.onnxruntime</groupId>
    <artifactId>onnxruntime_gpu</artifactId>
    <version>1.19.2</version>
</dependency>
```
