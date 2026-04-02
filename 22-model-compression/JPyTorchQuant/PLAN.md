# JPyTorchQuant — Migration Plan: Python PyTorch Quantization → Java 26

## Overview
PyTorch provides post-training and quantization-aware training via `torch.quantization`. Quantized models use INT8 operations for faster CPU inference with minimal accuracy loss.

## Java 26 Equivalent
**DJL** with TorchScript quantized models.

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

**Python (PyTorch Quantization):**
```python
import torch
model = MyModel()
model.qconfig = torch.quantization.get_default_qconfig('fbgemm')
torch.quantization.prepare(model, inplace=True)
# Calibrate with representative data
torch.quantization.convert(model, inplace=True)
torch.jit.save(torch.jit.script(model), 'quantized.pt')
```

**Java 26 (JPyTorchQuant):**
```java
import ai.djl.*;
import ai.djl.pytorch.engine.*;
// Load PyTorch quantized TorchScript model in DJL
Criteria<NDList, NDList> criteria = Criteria.builder()
    .setTypes(NDList.class, NDList.class)
    .optModelPath(Paths.get("quantized.pt"))
    .optEngine("PyTorch")
    .optOption("mapLocation", "cpu") // INT8 runs on CPU
    .build();
ZooModel<NDList, NDList> model = criteria.loadModel();
Predictor<NDList, NDList> predictor = model.newPredictor();
NDList result = predictor.predict(new NDList(inputTensor));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ai.djl.pytorch</groupId>
    <artifactId>pytorch-engine</artifactId>
    <version>0.28.0</version>
</dependency>
```
