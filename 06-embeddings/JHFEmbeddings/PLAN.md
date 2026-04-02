# JHFEmbeddings — Migration Plan: Python Hugging Face Embeddings → Java 26

## Overview
Hugging Face Embeddings API provides access to 10,000+ embedding models via the HF Inference API or local model loading. Popular models: bge-m3, E5-large, UAE-Large-V1, GTE-Qwen2.

## Java 26 Equivalent
**LangChain4j Hugging Face provider** and **DJL** embedding models.

## Why Java 26
- **Virtual Threads:** Async HTTP calls to HF Inference API without blocking threads.
- **Vector API:** SIMD-accelerate local embedding vector arithmetic.
- **Panama FFM:** Zero-copy tensor buffer exchange with local ONNX/PyTorch engine.
- **Structured Concurrency:** Parallel embedding requests with timeout and cancellation.

## GPU / TPU Support
- HF Inference API offloads GPU computation to Hugging Face servers.
- Local ONNX models via ONNX Runtime GPU Java for on-premises GPU inference.
- DJL with CUDA backend for local GPU embedding generation.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Remote: use LangChain4j `HuggingFaceChatModel` or `HuggingFaceEmbeddingModel`.
- Local: export to ONNX, load via `OnnxBertBiEncoder` in LangChain4j.
- Replace `hf_model.embed(text)` with `embeddingModel.embed(TextSegment.from(text))`.
- Set HF API token via `HuggingFaceEmbeddingModel.builder().accessToken(token)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Hugging Face Embeddings):**
```python
from huggingface_hub import InferenceClient
client = InferenceClient(token='hf_...')
embeddings = client.feature_extraction(
    'Hello world',
    model='BAAI/bge-m3'
)
print(len(embeddings[0])) # 1024
```

**Java 26 (JHFEmbeddings):**
```java
import dev.langchain4j.model.huggingface.*;
EmbeddingModel model = HuggingFaceEmbeddingModel.builder()
    .accessToken("hf_...")
    .modelId("BAAI/bge-m3")
    .waitForModel(true)
    .timeout(Duration.ofSeconds(30))
    .build();
Embedding emb = model.embed(TextSegment.from("Hello world")).content();
System.out.println(emb.vector().length); // 1024
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
    <artifactId>langchain4j-hugging-face</artifactId>
    <version>0.35.0</version>
</dependency>
```
