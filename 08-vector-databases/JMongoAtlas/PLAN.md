# JMongoAtlas — Migration Plan: Python MongoDB Atlas Search → Java 26

## Overview
MongoDB Atlas Search provides native vector search powered by Apache Lucene's HNSW implementation. It enables semantic search alongside MongoDB queries, transactions, and aggregation pipelines.

## Java 26 Equivalent
**MongoDB Java Driver** with Atlas Vector Search.

## Why Java 26
- **Virtual Threads:** Async MongoDB Java driver reactive streams integration.
- **Structured Concurrency:** Parallel insert and search operations.
- **Vector API:** Client-side embedding normalization before upsert.
- **Pattern Matching:** Aggregation pipeline stage type dispatch.

## GPU / TPU Support
- Atlas Vector Search runs HNSW on MongoDB Atlas clusters with NVMe SSD-backed nodes.
- GPU embedding generation (DJL CUDA) produces vectors uploaded to Atlas.
- Atlas dedicated clusters with high-memory nodes optimize large HNSW indexes.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `collection.insert_one({'embedding': vector})` with Java `collection.insertOne(doc)`.
- Replace Atlas `$vectorSearch` aggregation with Java driver `Aggregates.vectorSearch()`.
- Use `MongoClient.create(connectionString)` with Atlas connection string.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (MongoDB Atlas Search):**
```python
from pymongo import MongoClient
client = MongoClient('mongodb+srv://...')
col = client.mydb.docs
col.insert_one({'embedding': [0.1]*1536, 'text': 'hello'})
result = col.aggregate([{'$vectorSearch': {'index':'vs_idx','path':'embedding','queryVector':[0.1]*1536,'numCandidates':100,'limit':5}}])
```

**Java 26 (JMongoAtlas):**
```java
import com.mongodb.client.*;
import org.bson.*;
import com.mongodb.client.model.search.*;
MongoClient client = MongoClients.create("mongodb+srv://...");
MongoCollection<Document> col = client.getDatabase("mydb").getCollection("docs");
col.insertOne(new Document("embedding", List.of(0.1f, 0.2f)).append("text", "hello"));
List<Double> queryVec = List.of(0.1, 0.2, ...);
AggregateIterable<Document> results = col.aggregate(List.of(
    Aggregates.vectorSearch(FieldSearchPath.fieldPath("embedding"),
        queryVec, "vs_idx", 5, VectorSearchOptions.approximateVectorSearchOptions(100))));
results.forEach(d -> System.out.println(d.getString("text")));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>5.2.1</version>
</dependency>
```
