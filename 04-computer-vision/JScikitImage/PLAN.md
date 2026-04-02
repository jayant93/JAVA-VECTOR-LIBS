# JScikitImage — Migration Plan: Python scikit-image → Java 26

## Overview
scikit-image is a Python image processing library built on NumPy/SciPy providing algorithms for filtering, morphology, segmentation, feature extraction, and image restoration.

## Java 26 Equivalent
**BoofCV** — pure Java image processing and computer vision.

## Why Java 26
- **Vector API:** SIMD-accelerate convolution, morphological ops, and pixel statistics.
- **Panama FFM:** Zero-copy pixel data exchange with native image libraries.
- **Virtual Threads:** Concurrent processing of image batches.
- **Structured Concurrency:** Parallel filter application across image channels.

## GPU / TPU Support
- BoofCV can dispatch to OpenCV CUDA via JNI for GPU-accelerated filters.
- Custom CUDA kernels via Panama FFM for morphological operations.
- OpenCL-based image processing for AMD/Intel GPU acceleration.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `skimage.io.imread()` with BoofCV `UtilImageIO.loadImage()`.
- Replace `skimage.filters.gaussian()` with BoofCV `BlurImageOps.gaussian()`.
- Replace `skimage.feature.canny()` with BoofCV `CannyEdge`.
- Replace `skimage.morphology.binary_erosion()` with BoofCV `BinaryImageOps.erode()`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (scikit-image):**
```python
from skimage import io, filters, feature
img = io.imread('image.png', as_gray=True)
smoothed = filters.gaussian(img, sigma=2)
edges = feature.canny(smoothed, sigma=1)
io.imsave('edges.png', edges.astype('uint8') * 255)
```

**Java 26 (JScikitImage):**
```java
import boofcv.io.image.UtilImageIO;
import boofcv.alg.filter.blur.BlurImageOps;
import boofcv.alg.feature.detect.edge.CannyEdge;
import boofcv.struct.image.*;
GrayF32 img = UtilImageIO.loadImage("image.png", GrayF32.class);
GrayF32 smoothed = new GrayF32(img.width, img.height);
BlurImageOps.gaussian(img, smoothed, -1, 2, null);
GrayU8 edges = new GrayU8(img.width, img.height);
// Apply Canny edge detection
new CannyEdge<>(0.1f, 0.3f, true, GrayF32.class, GrayS16.class).process(smoothed, edges);
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
