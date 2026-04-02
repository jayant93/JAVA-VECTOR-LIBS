# JPillow — Migration Plan: Python Pillow (PIL) → Java 26

## Overview
Pillow is Python's standard image library for loading, saving, and transforming images. It supports 30+ formats (JPEG, PNG, TIFF, WebP) and common operations like crop, resize, rotate, and filter.

## Java 26 Equivalent
**ImageJ** and **BoofCV** for image loading and manipulation.

## Why Java 26
- **Panama FFM:** Direct pixel buffer manipulation via `MemorySegment` for zero-copy transforms.
- **Virtual Threads:** Batch image processing with concurrent I/O.
- **Vector API:** SIMD pixel operations (brightness, contrast, channel mixing).
- **Structured Concurrency:** Parallel image resize/convert pipelines.

## GPU / TPU Support
- GPU-accelerated image transforms via OpenCV CUDA in Java.
- DJL vision transforms use GPU-backed tensors for batch augmentation.
- NVIDIA DALI (accessible via subprocess/REST) for ultra-fast GPU image pipelines.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `Image.open(path)` with `ImageIO.read(new File(path))`.
- Replace `img.resize((w,h))` with `Imgproc.resize()` via OpenCV Java.
- Replace `img.convert('L')` (grayscale) with `Imgproc.cvtColor(..., COLOR_BGR2GRAY)`.
- Replace `img.save(path)` with `ImageIO.write(img, 'PNG', new File(path))`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Pillow (PIL)):**
```python
from PIL import Image
img = Image.open('photo.jpg')
resized = img.resize((224, 224))
gray = resized.convert('L')
gray.save('gray.png')
```

**Java 26 (JPillow):**
```java
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.awt.*;
BufferedImage img = ImageIO.read(new File("photo.jpg"));
Image scaled = img.getScaledInstance(224, 224, Image.SCALE_SMOOTH);
BufferedImage result = new BufferedImage(224, 224, BufferedImage.TYPE_BYTE_GRAY);
result.getGraphics().drawImage(scaled, 0, 0, null);
ImageIO.write(result, "PNG", new File("gray.png"));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.boofcv</groupId>
    <artifactId>boofcv-core</artifactId>
    <version>1.1.4</version>
</dependency>
```
