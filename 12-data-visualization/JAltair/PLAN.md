# JAltair — Migration Plan: Python Altair → Java 26

## Overview
Altair is a declarative statistical visualization library based on Vega-Lite. It uses a grammar of graphics approach where chart specifications are JSON-serializable Python objects.

## Java 26 Equivalent
**XChart** with declarative chart configuration.

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

**Python (Altair):**
```python
import altair as alt
import pandas as pd
df = pd.DataFrame({'x':[1,2,3],'y':[4,5,6]})
chart = alt.Chart(df).mark_point().encode(x='x',y='y')
chart.save('chart.html')
```

**Java 26 (JAltair):**
```java
import org.knowm.xchart.*;
// Declarative-style configuration in XChart
XYChart chart = new XYChartBuilder().width(600).height(400)
    .chartTheme(ChartTheme.GGPlot2).build();
chart.addSeries("points", new double[]{1,2,3}, new double[]{4,5,6})
    .setXYSeriesRenderStyle(XYSeries.XYSeriesRenderStyle.Scatter);
BitmapEncoder.saveBitmap(chart, "chart", BitmapEncoder.BitmapFormat.PNG);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.knowm.xchart</groupId>
    <artifactId>xchart</artifactId>
    <version>3.8.8</version>
</dependency>
```
