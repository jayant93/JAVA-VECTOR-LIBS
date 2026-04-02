# JPlotly — Migration Plan: Python Plotly → Java 26

## Overview
Plotly generates interactive web-based charts including 3D surface plots, sunbursts, and animated charts. The Plotly Express API simplifies common visualization patterns.

## Java 26 Equivalent
**Apache ECharts Java** for interactive web-based visualization.

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

**Python (Plotly):**
```python
import plotly.express as px
df = px.data.iris()
fig = px.scatter(df, x='sepal_width', y='sepal_length', color='species')
fig.write_html('scatter.html')
```

**Java 26 (JPlotly):**
```java
import org.apache.echarts.*;
// Apache ECharts Java: generate ECharts option JSON
Option option = new Option()
    .setTitle(new Title().setText("Iris Scatter"))
    .setSeries(new ScatterSeries().setData(scatterData));
String json = new Gson().toJson(option);
// Embed JSON in HTML with ECharts JS library
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.apache.echarts</groupId>
    <artifactId>echarts-java</artifactId>
    <version>1.0.5</version>
</dependency>
```
