# JOpenSearch — Migration Plan: Python OpenSearch (Python client) → Java 26

## Overview
OpenSearch is an open-source fork of Elasticsearch with native k-NN vector search via the k-NN plugin, supporting HNSW, IVF, and Faiss-backed indexes with GPU acceleration in 3.0+.

## Java 26 Equivalent
**OpenSearch Java Client** for k-NN vector search.

## Why Java 26
- **Virtual Threads:** Async HTTP/2 REST calls to OpenSearch cluster.
- **Structured Concurrency:** Parallel bulk index operations.
- **Vector API:** SIMD client-side embedding normalization.
- **Pattern Matching:** Query DSL builder type dispatch.

## GPU / TPU Support
- OpenSearch 3.0+ supports GPU-accelerated k-NN via CUDA-enabled Faiss.
- Enable GPU: `knn.algo_param.ef_search` tuning with GPU index type.
- Client sends float[] vectors; server-side GPU handles ANN computation.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `opensearchpy.OpenSearch(host)` with Java `OpenSearchClient(transport)`.
- Replace `client.index(index=name, body={...vector...})` with Java bulk index request.
- Replace `client.search(index=name, body={knn:{vector:q, k:10}})` with Java search DSL.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (OpenSearch (Python client)):**
```python
from opensearchpy import OpenSearch
client = OpenSearch('http://localhost:9200')
client.index(index='vecs', body={'embedding': [0.1]*128, 'text': 'hello'})
result = client.search(index='vecs', body={'query':{'knn':{'embedding':{'vector':[0.1]*128,'k':5}}}})
```

**Java 26 (JOpenSearch):**
```java
import org.opensearch.client.opensearch.*;
import org.opensearch.client.opensearch.core.*;
OpenSearchClient client = new OpenSearchClient(new RestClientTransport(
    RestClient.builder(new HttpHost("localhost", 9200)).build(),
    new JacksonJsonpMapper()));
// Index document with vector
client.index(r -> r.index("vecs").id("1")
    .document(Map.of("embedding", queryVector, "text", "hello")));
// kNN search
SearchResponse<Map> response = client.search(
    s -> s.index("vecs").query(q -> q.knn(k -> k.field("embedding").vector(queryVector).numCandidates(10))),
    (Class<Map>)(Class<?>)Map.class);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.opensearch.client</groupId>
    <artifactId>opensearch-java</artifactId>
    <version>2.15.0</version>
</dependency>
```
