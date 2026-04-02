# JDetectron2 — Migration Plan: Python Detectron2 → Java 26

## Overview
Detectron2 is Facebook AI's object detection and segmentation framework built on PyTorch. It supports Mask R-CNN, Faster R-CNN, RetinaNet, Panoptic FPN, and DensePose with strong extensibility.

## Java 26 Equivalent
**DJL** with PyTorch TorchScript engine for Detectron2 exported models.

## Why Java 26
- **Panama FFM:** Direct LibTorch binding for TorchScript model inference.
- **Virtual Threads:** Async image pre-processing and result post-processing.
- **Vector API:** SIMD-accelerate RoI pooling and feature map operations on CPU.
- **Structured Concurrency:** Parallel inference on batched image inputs.

## GPU / TPU Support
- DJL PyTorch engine supports CUDA — Detectron2 TorchScript models run on GPU.
- Export Detectron2 to TorchScript: `torch.jit.trace(model, inputs)` → `.pt` file.
- ONNX export and ONNX Runtime GPU for wider hardware support.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export model: `torch.jit.trace(model, sample_inputs).save('detectron2.pt')`.
- Load in DJL via PyTorch engine with `Criteria.optModelPath()`.
- Write custom `Translator` for image pre/post processing (NMS, mask decoding).
- Use `Predictor<Image, DetectedObjects>` for inference.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Detectron2):**
```python
from detectron2.engine import DefaultPredictor
from detectron2.config import get_cfg
cfg = get_cfg()
cfg.MODEL.WEIGHTS = 'detectron2://COCO-Detection/faster_rcnn_R_50_FPN_3x/137849458/model_final_280758.pkl'
predictor = DefaultPredictor(cfg)
outputs = predictor(image)
```

**Java 26 (JDetectron2):**
```java
import ai.djl.*;
import ai.djl.modality.cv.*;
// Load exported TorchScript Detectron2 model
Criteria<Image, DetectedObjects> criteria = Criteria.builder()
    .setTypes(Image.class, DetectedObjects.class)
    .optModelPath(Paths.get("detectron2.pt"))
    .optEngine("PyTorch")
    .optTranslator(new MyDetectron2Translator())
    .build();
ZooModel<Image, DetectedObjects> model = criteria.loadModel();
DetectedObjects result = model.newPredictor().predict(image);
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
