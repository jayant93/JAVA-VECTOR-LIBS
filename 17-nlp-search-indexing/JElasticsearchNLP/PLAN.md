# JElasticsearchNLP — Migration Plan: Python Elasticsearch (NLP) → Java 26

## Overview
Elasticsearch supports NLP via its ML node inference pipeline: text embeddings, named entity recognition, text classification, and fill-mask using transformer models deployed to ML nodes.

## Java 26 Equivalent
**Elasticsearch Java API Client** with ML node NLP inference.

## Why Java 26
- **Virtual Threads:** Concurrent document indexing pipelines.
- **Vector API:** SIMD BM25 scoring computation.
- **Structured Concurrency:** Parallel shard search with result merging.
- **Pattern Matching:** Query type dispatch (match, phrase, wildcard).

## GPU / TPU Support
Lucene Vector API (Java SIMD) accelerates HNSW and distance computation. GPU embedding generation feeds into hybrid keyword+vector search. GPU-accelerated Lucene planned for future releases.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Build Lucene index with appropriate analyzers.
- Replace Python query execution with Lucene `IndexSearcher.search()`.
- Combine BM25 with vector search using `BooleanQuery` + `KnnVectorQuery`.
- Use Lucene's built-in BM25Similarity as default scorer.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Elasticsearch (NLP)):**
```python
from elasticsearch import Elasticsearch
es = Elasticsearch()
result = es.ml.infer_trained_model(model_id='my_ner_model', docs=[{'text_field':'Apple is a company'}])
print(result['inference_results'][0]['predicted_value'])
```

**Java 26 (JElasticsearchNLP):**
```java
import co.elastic.clients.elasticsearch.*;
import co.elastic.clients.elasticsearch.ml.*;
ElasticsearchClient client = ...; // initialized client
InferTrainedModelResponse resp = client.ml().inferTrainedModel(r -> r
    .modelId("my_ner_model")
    .docs(List.of(InferenceConfigCreateContainer.of(c ->
        c.textClassification(t -> t.numTopClasses(1)))))
    .build());
System.out.println(resp.inferenceResults());
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
