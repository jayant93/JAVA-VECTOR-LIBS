# JWhoosh — Migration Plan: Python Whoosh → Java 26

## Overview
Whoosh is a pure Python full-text search library built for simplicity. It provides in-process indexing and search with BM25F scoring, field analysis, and faceted search.

## Java 26 Equivalent
**Apache Lucene** for full-text search and indexing in Java.

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

**Python (Whoosh):**
```python
from whoosh import index, fields, qparser
schema = fields.Schema(title=fields.TEXT, content=fields.TEXT)
ix = index.create_in('indexdir', schema)
writer = ix.writer()
writer.add_document(title='Doc 1', content='Java is great')
writer.commit()
```

**Java 26 (JWhoosh):**
```java
import org.apache.lucene.document.*;
import org.apache.lucene.index.*;
import org.apache.lucene.store.*;
FSDirectory dir = FSDirectory.open(Path.of("indexdir"));
IndexWriter writer = new IndexWriter(dir, new IndexWriterConfig(new StandardAnalyzer()));
Document doc = new Document();
doc.add(new TextField("title", "Doc 1", Field.Store.YES));
doc.add(new TextField("content", "Java is great", Field.Store.YES));
writer.addDocument(doc);
writer.commit(); writer.close();
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>9.12.0</version>
</dependency>
```
