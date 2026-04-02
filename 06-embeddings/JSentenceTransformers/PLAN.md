# JSentenceTransformers — Migration Plan: Python Sentence-Transformers → Java 26

## Overview
Sentence-Transformers generates dense vector embeddings from text using fine-tuned transformer models (BERT, RoBERTa, MPNet). It is the standard library for semantic similarity, clustering, and retrieval.

## Java 26 Equivalent
**DJL Embeddings** and **LangChain4j** embedding providers.

## Why Java 26
- **Panama FFM:** Zero-copy tensor transfer between tokenizer and transformer model.
- **Virtual Threads:** Concurrent batch encoding of large document collections.
- **Vector API:** SIMD-accelerate cosine similarity computation over embedding vectors.
- **Structured Concurrency:** Fan-out parallel encoding across multiple model replicas.

## GPU / TPU Support
- DJL encodes embeddings using GPU-backed PyTorch/TF engines via CUDA.
- ONNX-exported sentence-transformer models run on GPU via ONNX Runtime Java.
- Multi-GPU throughput via DJL serving with device-parallel predictor pool.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export model to ONNX: `model.save_pretrained('model_onnx', safe_serialization=False)` + Optimum.
- Load in LangChain4j: `new OnnxBertBiEncoder(...)` or `new HuggingFaceEmbeddingModel(...)`.
- Replace `model.encode(sentences)` with `embeddingModel.embedAll(textSegments)`.
- Cosine similarity: use `CosineSimilarity.between(emb1, emb2)` in LangChain4j.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Sentence-Transformers):**
```python
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('all-MiniLM-L6-v2')
sentences = ['Hello world', 'Java is great for AI']
embeddings = model.encode(sentences)
print(embeddings.shape) # (2, 384)
```

**Java 26 (JSentenceTransformers):**
```java
import dev.langchain4j.model.embedding.*;
import dev.langchain4j.model.embedding.onnx.allminilml6v2.*;
EmbeddingModel model = new AllMiniLmL6V2EmbeddingModel();
List<TextSegment> segments = List.of(
    TextSegment.from("Hello world"),
    TextSegment.from("Java is great for AI"));
Response<List<Embedding>> response = model.embedAll(segments);
System.out.println(response.content().get(0).vector().length); // 384
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-embeddings-all-minilm-l6-v2</artifactId>
    <version>0.35.0</version>
</dependency>
```
