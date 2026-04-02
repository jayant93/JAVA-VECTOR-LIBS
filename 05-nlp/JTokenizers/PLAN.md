# JTokenizers — Migration Plan: Python HuggingFace Tokenizers → Java 26

## Overview
HuggingFace Tokenizers is a Rust-based tokenization library providing fast BPE, WordPiece, and Unigram tokenizers used in all modern LLMs. It is 10-100x faster than pure Python tokenizers.

## Java 26 Equivalent
**HuggingFace Tokenizers Java** (Rust-based, via DJL bindings).

## Why Java 26
- **Panama FFM:** Direct Rust tokenizer library binding via FFM for zero-JNI overhead.
- **Virtual Threads:** Concurrent tokenization of thousands of documents.
- **Vector API:** SIMD-accelerate vocabulary lookup and token ID mapping.
- **Structured Concurrency:** Batch tokenization with parallel encoding.

## GPU / TPU Support
- Tokenization is CPU-bound; GPU not needed for tokenization itself.
- Tokenized output (input_ids, attention_mask) passed directly to GPU model via CUDA tensor.
- Pinned memory transfers via Panama FFM for fast CPU→GPU token tensor copies.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Add DJL `huggingface-tokenizers` dependency.
- Replace `AutoTokenizer.from_pretrained(name)` with `HuggingFaceTokenizer.newInstance(name)`.
- Replace `tokenizer(text, return_tensors='pt')` with `tokenizer.encode(text)` → `Encoding`.
- Access `encoding.getIds()`, `encoding.getAttentionMask()`, `encoding.getTokenTypeIds()`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (HuggingFace Tokenizers):**
```python
from transformers import AutoTokenizer
tokenizer = AutoTokenizer.from_pretrained('bert-base-uncased')
enc = tokenizer('Hello, world!', return_tensors='pt')
print(enc['input_ids'])
print(enc['attention_mask'])
```

**Java 26 (JTokenizers):**
```java
import ai.djl.huggingface.tokenizers.*;
HuggingFaceTokenizer tokenizer = HuggingFaceTokenizer.newInstance(
    "bert-base-uncased", Map.of("maxLength", "512", "padding", "true"));
Encoding enc = tokenizer.encode("Hello, world!");
System.out.println(Arrays.toString(enc.getIds()));
System.out.println(Arrays.toString(enc.getAttentionMask()));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ai.djl</groupId>
    <artifactId>huggingface-tokenizers</artifactId>
    <version>0.28.0</version>
</dependency>
```
