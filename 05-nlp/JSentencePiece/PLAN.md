# JSentencePiece — Migration Plan: Python SentencePiece → Java 26

## Overview
SentencePiece is an unsupervised text tokenizer implementing BPE and Unigram language model tokenization. It is used in modern LLMs (T5, LLaMA, Gemma, Mistral) for language-agnostic subword tokenization.

## Java 26 Equivalent
**DJL NLP SentencePiece** bindings and **OpenNLP** tokenizers.

## Why Java 26
- **Panama FFM:** Direct SentencePiece C++ library binding for maximum throughput.
- **Virtual Threads:** Concurrent tokenization of document batches.
- **Vector API:** SIMD-accelerate vocabulary trie lookup.
- **Structured Concurrency:** Parallel encode/decode across document shards.

## GPU / TPU Support
- SentencePiece tokenization is CPU-bound; output feeds directly into GPU model inference.
- Fast CPU tokenization combined with CUDA inference provides optimal end-to-end throughput.
- Pinned memory allocation via Panama FFM for fast CPU→GPU token transfer.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Use DJL's `HuggingFaceTokenizer` which supports SentencePiece models internally.
- Load `.model` file: `SentencePieceProcessor.getOrCreate(modelPath)`.
- Replace `sp.encode(text, out_type=str)` with `tokenizer.encode(text).getTokens()`.
- Replace `sp.decode(ids)` with `tokenizer.decode(ids)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (SentencePiece):**
```python
import sentencepiece as spm
sp = spm.SentencePieceProcessor()
sp.load('tokenizer.model')
tokens = sp.encode('Hello world', out_type=str)
ids = sp.encode('Hello world', out_type=int)
print(tokens, ids)
```

**Java 26 (JSentencePiece):**
```java
import ai.djl.huggingface.tokenizers.*;
// SentencePiece-based tokenizer (LLaMA, T5, etc.)
HuggingFaceTokenizer tokenizer = HuggingFaceTokenizer.newInstance(
    Paths.get("tokenizer.model"), Map.of());
Encoding enc = tokenizer.encode("Hello world");
System.out.println(Arrays.toString(enc.getTokens()));
System.out.println(Arrays.toString(enc.getIds()));
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
