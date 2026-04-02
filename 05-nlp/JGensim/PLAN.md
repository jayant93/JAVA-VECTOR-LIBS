# JGensim — Migration Plan: Python Gensim → Java 26

## Overview
Gensim provides Word2Vec, FastText, Doc2Vec for word/document embeddings and LDA/LSI for topic modeling. It is widely used for training custom embeddings on domain-specific corpora.

## Java 26 Equivalent
**Word2Vec Java** (deeplearning4j) and **Smile** for topic modeling.

## Why Java 26
- **Virtual Threads:** Parallel corpus scanning and window-based training.
- **Vector API:** SIMD-accelerate skip-gram negative sampling computation.
- **Structured Concurrency:** Parallel training epochs across corpus shards.
- **Pattern Matching:** Clean dispatch for different model types (Word2Vec vs. FastText).

## GPU / TPU Support
- GPU training available via DL4J Word2Vec CUDA backend for large corpora.
- ND4J CUDA backend for fast matrix operations in embedding training.
- ONNX export of trained embeddings for GPU-accelerated downstream inference.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `gensim.models.Word2Vec(sentences)` with DL4J `Word2Vec.Builder().iterate(iter).build()`.
- Replace `model.wv['word']` with `word2vec.getWordVectorMatrix('word')`.
- Replace `model.wv.most_similar('word')` with `word2vec.wordsNearest('word', n)`.
- For LDA, use Smile `LDA` or MALLET via Java subprocess.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Gensim):**
```python
from gensim.models import Word2Vec
sentences = [['machine','learning','is','fun'], ['deep','learning','models']]
model = Word2Vec(sentences, vector_size=100, window=5, min_count=1)
vec = model.wv['learning']
print(model.wv.most_similar('learning', topn=3))
```

**Java 26 (JGensim):**
```java
import org.deeplearning4j.models.word2vec.Word2Vec;
import org.deeplearning4j.text.sentenceiterator.BasicLineIterator;
import org.deeplearning4j.text.tokenization.tokenizerfactory.DefaultTokenizerFactory;
Word2Vec w2v = new Word2Vec.Builder()
    .minWordFrequency(1).iterations(5).layerSize(100).windowSize(5)
    .iterate(new BasicLineIterator("corpus.txt"))
    .tokenizerFactory(new DefaultTokenizerFactory()).build();
w2v.fit();
System.out.println(w2v.wordsNearest("learning", 3));
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
