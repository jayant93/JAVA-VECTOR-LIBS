# JSolr — Migration Plan: Python Apache Solr → Java 26

## Overview
Apache Solr is a production-proven enterprise search platform built on Lucene. It provides full-text search, faceting, distributed search (SolrCloud), and dense vector search with HNSW indexes.

## Java 26 Equivalent
**Apache Solr Java SolrJ client** for enterprise search.

## Why Java 26
- **Virtual Threads:** Async SolrJ HTTP calls to Solr cluster.
- **Structured Concurrency:** Parallel shard queries with result merging.
- **Vector API:** SIMD-accelerate client-side embedding normalization.
- **Pattern Matching:** SolrQuery builder type dispatch.

## GPU / TPU Support
Solr HNSW vector search uses Lucene's Vector API (SIMD). GPU embedding generation client-side feeds Solr. Solr 9.6+ supports neural sparse vectors via SPLADE.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Add SolrJ dependency to Maven.
- Replace `solr.add(docs)` with Java `client.add(SolrInputDocument)` via `HttpSolrClient`.
- Replace vector search with Java `SolrQuery.setQuery('{!knn f=embedding topK=10}...')`.
- Replace Python `results.docs` with Java `QueryResponse.getResults()`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Apache Solr):**
```python
import pysolr
solr = pysolr.Solr('http://localhost:8983/solr/mycore')
solr.add([{'id':'1','text':'hello','embedding':'[0.1 0.2 ...]'}])
results = solr.search('{!knn f=embedding topK=5}[0.1 0.2 ...]')
for r in results: print(r['text'])
```

**Java 26 (JSolr):**
```java
import org.apache.solr.client.solrj.*;
import org.apache.solr.client.solrj.impl.*;
import org.apache.solr.common.*;
SolrClient client = new HttpSolrClient.Builder("http://localhost:8983/solr/mycore").build();
SolrInputDocument doc = new SolrInputDocument();
doc.addField("id","1"); doc.addField("text","hello");
doc.addField("embedding", new float[]{0.1f, 0.2f, ...});
client.add(doc); client.commit();
QueryResponse resp = client.query(new SolrQuery("{!knn f=embedding topK=5}[0.1 0.2]"));
resp.getResults().forEach(r -> System.out.println(r.get("text")));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-solrj</artifactId>
    <version>9.7.0</version>
</dependency>
```
