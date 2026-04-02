# JRedisSearch — Migration Plan: Python Redis + RedisSearch → Java 26

## Overview
Redis with RedisSearch extension supports vector similarity search (FLAT and HNSW indexes) alongside full-text search and JSON storage, enabling sub-millisecond hybrid search at scale.

## Java 26 Equivalent
**Jedis** (Redis Java client) with RediSearch/Redis Stack vector search.

## Why Java 26
- **Virtual Threads:** High-concurrency pipelined Redis commands.
- **Structured Concurrency:** Parallel batch vector inserts with timeout control.
- **Vector API:** Client-side SIMD vector normalization before HSET.
- **Pattern Matching:** Query filter expression building.

## GPU / TPU Support
- Redis in-memory HNSW index: pure CPU-based, but very fast due to memory access patterns.
- GPU-accelerated embedding generation (DJL CUDA) feeds into Redis vector store.
- Redis Enterprise with GPU-enabled nodes for larger vector collections.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `redis.hset(key, mapping={...vector...})` with Jedis `client.hset(key, fields)`.
- Replace `redis.ft().search(index, query)` with Jedis `client.ftSearch(index, query)`.
- Use `VectorFieldAttributes.HNSW` for index creation in Jedis.
- Replace `@dataclass` schema with Jedis `Schema.addHNSWVectorField()`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Redis + RedisSearch):**
```python
import redis
from redis.commands.search.field import VectorField
r = redis.Redis()
pipe = r.pipeline()
pipe.hset('vec:1', mapping={'vector': np.array([0.1]*128).tobytes(), 'text': 'hello'})
pipe.execute()
result = r.ft('idx').search('*=>[KNN 5 @vector $query]', query_params={'query': query_bytes})
```

**Java 26 (JRedisSearch):**
```java
import redis.clients.jedis.*;
import redis.clients.jedis.search.*;
JedisPooled client = new JedisPooled("localhost", 6379);
// Store vector as byte array
byte[] vecBytes = floatArrayToBytes(new float[]{0.1f, 0.2f, ...});
client.hset("vec:1", Map.of("vector", new String(vecBytes), "text", "hello"));
// KNN search
Query q = new Query("*=>[KNN 5 @vector $query AS score]")
    .addParam("query", queryBytes).returnFields("text", "score").dialect(2);
SearchResult result = client.ftSearch("idx", q);
result.getDocuments().forEach(d -> System.out.println(d.getString("text")));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>5.2.0</version>
</dependency>
```
