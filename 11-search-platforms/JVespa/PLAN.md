# JVespa — Migration Plan: Python Vespa → Java 26

## Overview
Vespa is Yahoo's open-source data serving engine that combines vector search, traditional search, and ML-based ranking. It handles real-time indexing and serves massive datasets across distributed nodes.

## Java 26 Equivalent
**Vespa Java-native** (Vespa is itself a Java-based system).

## Why Java 26
- **Virtual Threads:** Async Vespa document feed and query API calls.
- **Structured Concurrency:** Parallel content cluster shard queries.
- **Vector API:** SIMD distance computation for custom ranking expressions.
- **Pattern Matching:** Document schema type dispatch in Java components.

## GPU / TPU Support
Vespa supports GPU-accelerated tensor computation via its own tensor framework. Vespa ML ranking models run on GPU-enabled content nodes. ONNX models deployed to Vespa evaluate on GPU.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Use Vespa Java document API for feeding: `DocumentFeed` or `VespaFeed`.
- Replace Python `vespa.query(yql=...)` with Vespa Java client `target.query(queryRequest)`.
- Replace document schema class with Vespa `Document` with field definitions.
- Deploy ONNX models to Vespa application package for server-side GPU ranking.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Vespa):**
```python
from vespa.application import Vespa
app = Vespa(url='http://localhost:8080')
app.feed_data_point(schema='doc', data_id='1', fields={'text':'hello','embedding':[0.1]*128})
result = app.query({'yql':'select * from doc where ({targetHits:5}nearestNeighbor(embedding,query_emb))','query-tensor-type':'tensor<float>(x[128])','query-embedding':[0.1]*128})
```

**Java 26 (JVespa):**
```java
import com.yahoo.vespa.client.*;
// Vespa Java components (searcher, document processor)
import com.yahoo.search.*;
public class MySearcher extends Searcher {
    @Override
    public Result search(Query query, Execution execution) {
        query.getModel().setRestrict("doc");
        query.properties().set("ranking.features.query(q_emb)", queryTensor);
        return execution.search(query);
    }
}
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<!-- Vespa is Java-native; use Vespa HTTP feeding API -->
<dependency>
    <groupId>com.yahoo.vespa</groupId>
    <artifactId>vespa-feed-client</artifactId>
    <version>8.433.21</version>
</dependency>
```
