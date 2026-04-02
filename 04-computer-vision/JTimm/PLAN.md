# JTimm — Migration Plan: Python timm → Java 26

## Overview
timm (PyTorch Image Models) by Ross Wightman provides 1000+ pre-trained vision models — EfficientNet, ViT, ConvNeXt, Swin Transformer, DeiT, ResNet — all with consistent training and fine-tuning API.

## Java 26 Equivalent
**DJL Model Zoo** and **ONNX Runtime Java** for 1000+ pre-trained vision models.

## Why Java 26
- **Virtual Threads:** Parallel model loading and inference across image batches.
- **Panama FFM:** Direct tensor memory sharing between Java and native model runtime.
- **Structured Concurrency:** Fan-out batch inference across available GPU devices.
- **Vector API:** SIMD-accelerate softmax and top-k post-processing on CPU.

## GPU / TPU Support
- Export timm model to ONNX: `torch.onnx.export(model, sample, 'model.onnx')`.
- Run via ONNX Runtime Java with CUDA execution provider.
- DJL Model Zoo provides ResNet, EfficientNet, and ViT variants natively.
- TensorRT optimization via ONNX Runtime TensorRT provider for maximum GPU throughput.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export desired timm model to ONNX from Python.
- Load in Java via ONNX Runtime: `OrtSession` with CUDA provider.
- Pre-process: resize → center crop → normalize to ImageNet mean/std.
- Post-process: softmax on logits → top-k class labels.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (timm):**
```python
import timm, torch
model = timm.create_model('efficientnet_b0', pretrained=True).eval()
data_config = timm.data.resolve_model_data_config(model)
transforms = timm.data.create_transform(**data_config, is_training=False)
output = model(transforms(img).unsqueeze(0))
```

**Java 26 (JTimm):**
```java
import com.microsoft.onnxruntime.*;
OrtEnvironment env = OrtEnvironment.getEnvironment();
OrtSession session = env.createSession("efficientnet_b0.onnx",
    new OrtSession.SessionOptions().addCUDA(0));
float[][][][] input = preprocessImageNet("img.jpg"); // resize+crop+normalize
OnnxTensor tensor = OnnxTensor.createTensor(env, input);
OrtSession.Result out = session.run(Map.of("input", tensor));
float[][] logits = (float[][]) out.get(0).getValue();
// Apply softmax and get top-5 predictions
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
