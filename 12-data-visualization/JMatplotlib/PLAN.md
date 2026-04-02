# JMatplotlib — Migration Plan: Python Matplotlib → Java 26

## Overview
Matplotlib is Python's foundational 2D plotting library for line, scatter, bar, histogram, and heatmap charts. It is the standard for scientific visualization and ML training metrics.

## Java 26 Equivalent
**JFreeChart** for Java charting.

## Why Java 26
- **Virtual Threads:** Async chart rendering and file export.
- **Vector API:** SIMD-accelerate data binning and aggregation.
- **Structured Concurrency:** Parallel chart generation for dashboards.
- **Pattern Matching:** Chart series type dispatch.

## GPU / TPU Support
Visualization is CPU-bound rendering. GPU not typically used. GPU-accelerated rendering possible via LWJGL/OpenGL for real-time 3D visualization in Java.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace Python plotting calls with JFreeChart/XChart equivalents.
- Replace `plt.show()` with swing window or file export.
- Replace pandas DataFrames with Tablesaw or double[][] arrays.
- Export charts to PNG/HTML for embedding in reports.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Matplotlib):**
```python
import matplotlib.pyplot as plt
plt.plot([1,2,3,4],[1,4,9,16])
plt.title('Square numbers')
plt.xlabel('x'); plt.ylabel('x²')
plt.savefig('chart.png')
```

**Java 26 (JMatplotlib):**
```java
import org.jfree.chart.*;
import org.jfree.data.xy.*;
XYDataset dataset = new XYSeriesCollection(new XYSeries("Squares"){{
    add(1,1); add(2,4); add(3,9); add(4,16);}});
JFreeChart chart = ChartFactory.createXYLineChart("Square numbers","x","x²",dataset);
ChartUtils.saveChartAsPNG(new File("chart.png"), chart, 800, 600);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.jfree</groupId>
    <artifactId>jfreechart</artifactId>
    <version>1.5.5</version>
</dependency>
```
