# JYOLOv8 — Migration Plan: Python YOLOv8 → Java 26

## Overview
YOLOv8 by Ultralytics is the state-of-the-art real-time object detection model family. It offers multiple sizes (nano to xlarge) for detection, segmentation, pose estimation, and classification.

## Java 26 Equivalent
**DJL Model Zoo** with YOLO ONNX model loaded via ONNX Runtime Java.

## Why Java 26
- **Virtual Threads:** Async frame grabbing and inference pipelining for video streams.
- **Panama FFM:** Direct NMS (Non-Maximum Suppression) C++ binding for post-processing.
- **Structured Concurrency:** Parallel pre-processing and post-processing of batches.
- **Vector API:** SIMD-accelerate bounding box IoU computation.

## GPU / TPU Support
- Export YOLOv8 to ONNX: `model.export(format='onnx')` in Python.
- Run via ONNX Runtime Java with CUDA execution provider for GPU inference.
- TensorRT-optimized engine via Triton Inference Server Java client for maximum GPU throughput.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export YOLOv8 model to ONNX format from Python.
- Load in Java via ONNX Runtime: `OrtEnvironment` + `OrtSession`.
- Pre-process image to float32 tensor [1,3,640,640], normalize to [0,1].
- Post-process output: apply confidence threshold + NMS on bounding boxes.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (YOLOv8):**
```python
from ultralytics import YOLO
model = YOLO('yolov8n.pt')
results = model('image.jpg')
for r in results:
    print(r.boxes.xyxy, r.boxes.conf, r.boxes.cls)
```

**Java 26 (JYOLOv8):**
```java
import ai.onnxruntime.*;
OrtEnvironment env = OrtEnvironment.getEnvironment();
OrtSession session = env.createSession("yolov8n.onnx",
    new OrtSession.SessionOptions()
        .addCUDA(0)); // GPU device 0
float[][][][] input = preprocessImage("image.jpg"); // [1,3,640,640]
OnnxTensor tensor = OnnxTensor.createTensor(env, input);
OrtSession.Result result = session.run(Map.of("images", tensor));
float[][][] output = (float[][][]) result.get(0).getValue();
// Apply NMS post-processing...
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
