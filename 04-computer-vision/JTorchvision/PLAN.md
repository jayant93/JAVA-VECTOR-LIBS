# JTorchvision — Migration Plan: Python torchvision → Java 26

## Overview
torchvision provides datasets, pre-trained vision models (ResNet, EfficientNet, ViT), and image transforms for PyTorch. It is the standard toolkit for transfer learning in computer vision.

## Java 26 Equivalent
**DJL Vision module** with pre-trained models and transforms.

## Why Java 26
- **Virtual Threads:** Concurrent image loading and augmentation during training.
- **Vector API:** SIMD-accelerate normalization and color jitter transforms.
- **Panama FFM:** Zero-copy tensor-to-image buffer conversion.
- **Structured Concurrency:** Parallel data loading workers (replaces num_workers).

## GPU / TPU Support
- DJL Vision uses PyTorch/TF CUDA engines — all inference runs on GPU.
- GPU-accelerated transforms via DJL's `NDArray` on CUDA backend.
- TPU inference via TF Java API for TorchVision-equivalent TF Hub models.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Load DJL Model Zoo pre-trained ResNet/EfficientNet via `Criteria` builder.
- Replace `transforms.Compose([...])` with DJL `Pipeline` of `Transform` objects.
- Replace `torchvision.models.resnet50(pretrained=True)` with DJL Model Zoo `ResNet50`.
- Replace `DataLoader(dataset, batch_size=32)` with DJL `RandomAccessDataset`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (torchvision):**
```python
import torchvision.models as models
import torchvision.transforms as T
model = models.resnet50(pretrained=True).eval()
transform = T.Compose([T.Resize(256), T.CenterCrop(224), T.ToTensor()])
output = model(transform(img).unsqueeze(0))
```

**Java 26 (JTorchvision):**
```java
import ai.djl.modality.cv.*;
import ai.djl.modality.cv.transform.*;
import ai.djl.repository.zoo.*;
Criteria<Image, Classifications> criteria = Criteria.builder()
    .optApplication(Application.CV.IMAGE_CLASSIFICATION)
    .setTypes(Image.class, Classifications.class)
    .optFilter("backbone", "resnet50")
    .build();
ZooModel<Image, Classifications> model = criteria.loadModel();
Predictor<Image, Classifications> p = model.newPredictor();
System.out.println(p.predict(ImageFactory.getInstance().fromFile(Paths.get("img.jpg"))));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ai.djl</groupId>
    <artifactId>model-zoo</artifactId>
    <version>0.28.0</version>
</dependency>
```
