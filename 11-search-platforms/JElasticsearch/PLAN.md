# JElasticsearch — Migration Plan: Python Elasticsearch → Java 26

## Overview
Elasticsearch is the world's most popular search engine, built on Apache Lucene. It provides full-text search, vector search (dense and sparse), aggregations, and real-time analytics at scale.

## Java 26 Equivalent
**Elasticsearch Java API Client** (official).

## Why Java 26
- **Virtual Threads:** Async REST/HTTP/2 calls to Elasticsearch cluster.
- **Structured Concurrency:** Parallel bulk indexing with error handling.
- **Vector API:** Client-side SIMD embedding normalization.
- **Pattern Matching:** Query DSL builder type dispatch.

## GPU / TPU Support
Elasticsearch HNSW vector search uses AVX-512 SIMD server-side. GPU embedding generation client-side via DJL CUDA feeds into Elasticsearch. ES 9.x planned GPU-accelerated inference via ML nodes.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Add Elasticsearch Java client dependency.
- Replace `es.index(index=name, document=doc)` with Java `client.index(r -> r.index(name).document(doc))`.
- Replace kNN search with Java `client.search(s -> s.knn(k -> k.field('vector').queryVector(v).k(10).numCandidates(100)))`.
- Replace Python scroll API with Java `OpenPointInTimeRequest` + search after.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Elasticsearch):**
```python
from elasticsearch import Elasticsearch
es = Elasticsearch('http://localhost:9200')
es.index(index='docs', document={'text':'hello','embedding':[0.1]*1536})
result = es.search(index='docs', knn={'field':'embedding','query_vector':[0.1]*1536,'k':5,'num_candidates':100})
```

**Java 26 (JElasticsearch):**
```java
import co.elastic.clients.elasticsearch.*;
import co.elastic.clients.elasticsearch.core.*;
ElasticsearchClient client = new ElasticsearchClient(
    new RestClientTransport(RestClient.builder(new HttpHost("localhost", 9200)).build(),
    new JacksonJsonpMapper()));
client.index(r -> r.index("docs").id("1").document(new DocRecord("hello", queryVec)));
SearchResponse<DocRecord> response = client.search(s -> s
    .index("docs")
    .knn(k -> k.field("embedding").queryVector(queryVec).k(5).numCandidates(100)),
    DocRecord.class);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>8.15.3</version>
</dependency>
```
