# JWord2Vec — Migration Plan: Python Gensim Word2Vec/FastText → Java 26

## Overview
Gensim's Word2Vec and FastText train word embeddings from raw text corpora. Word2Vec uses skip-gram/CBOW; FastText extends it with subword information, enabling embeddings for out-of-vocabulary words.

## Java 26 Equivalent
**DL4J Word2Vec** and **FastText4j** for word embedding training.

## Why Java 26
- **Virtual Threads:** Parallel corpus streaming and sliding-window processing.
- **Vector API:** SIMD-accelerate negative sampling dot products during training.
- **Structured Concurrency:** Fan-out training across corpus shards.
- **Panama FFM:** Call native FastText library (C++) for high-performance training.

## GPU / TPU Support
- ND4J CUDA backend accelerates embedding matrix updates during training.
- DL4J Word2Vec uses GPU-backed ND4J for large-scale embedding training.
- FastText C++ via Panama FFM binding for native-speed training on GPU.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `gensim.models.Word2Vec(corpus, vector_size=100)` with DL4J `Word2Vec.Builder().layerSize(100)`.
- For FastText: use `gensim.models.FastText` → DL4J `FastText` class or native binding.
- Replace `model.wv.most_similar('king')` with `word2vec.wordsNearest('king', 5)`.
- Load pre-trained vectors: `WordVectorSerializer.loadStaticModel(file)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Gensim Word2Vec/FastText):**
```python
from gensim.models import Word2Vec, FastText
w2v = Word2Vec([['the','quick','brown','fox']], vector_size=100, window=5)
ft = FastText([['the','quick','brown','fox']], vector_size=100)
print(w2v.wv.most_similar('fox', topn=3))
```

**Java 26 (JWord2Vec):**
```java
import org.deeplearning4j.models.word2vec.Word2Vec;
import org.deeplearning4j.models.embeddings.wordvectors.WordVectors;
Word2Vec w2v = new Word2Vec.Builder()
    .layerSize(100).windowSize(5).minWordFrequency(1)
    .iterate(new CollectionSentenceIterator(sentences))
    .tokenizerFactory(new DefaultTokenizerFactory()).build();
w2v.fit();
Collection<String> nearest = w2v.wordsNearest("fox", 3);
System.out.println(nearest);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>org.deeplearning4j</groupId>
    <artifactId>deeplearning4j-nlp</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```
