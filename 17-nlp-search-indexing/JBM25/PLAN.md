# JBM25 — Migration Plan: Python BM25 (rank-bm25) → Java 26

## Overview
BM25 (Best Match 25) is the standard probabilistic relevance ranking algorithm used in most search engines. It improves on TF-IDF with document length normalization.

## Java 26 Equivalent
**Apache Lucene BM25** — built-in similarity algorithm.

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

**Python (BM25 (rank-bm25)):**
```python
from rank_bm25 import BM25Okapi
corpus = [doc.split() for doc in documents]
bm25 = BM25Okapi(corpus)
scores = bm25.get_scores(query.split())
top_n = bm25.get_top_n(query.split(), documents, n=5)
```

**Java 26 (JBM25):**
```java
import org.apache.lucene.search.similarities.*;
import org.apache.lucene.index.*;
import org.apache.lucene.search.*;
// Lucene uses BM25 by default (BM25Similarity)
IndexSearcher searcher = new IndexSearcher(DirectoryReader.open(dir));
searcher.setSimilarity(new BM25Similarity(1.2f, 0.75f)); // k1, b params
QueryParser parser = new QueryParser("content", new StandardAnalyzer());
TopDocs results = searcher.search(parser.parse("search query"), 5);
for (ScoreDoc sd : results.scoreDocs)
    System.out.println(searcher.doc(sd.doc).get("text") + " score=" + sd.score);
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
    <artifactId>lucene-core</artifactId>
    <version>9.12.0</version>
</dependency>
```
