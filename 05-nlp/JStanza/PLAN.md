# JStanza — Migration Plan: Python Stanza (StanfordNLP) → Java 26

## Overview
Stanza (previously StanfordNLP) is Stanford's Python NLP library providing neural models for tokenization, POS tagging, NER, dependency parsing, and coreference resolution in 70+ languages.

## Java 26 Equivalent
**Stanford CoreNLP** (the Java implementation Stanza is based on).

## Why Java 26
- **Virtual Threads:** Concurrent document processing across language pipelines.
- **Vector API:** SIMD-accelerate neural scoring in transition-based parsing.
- **Structured Concurrency:** Parallel annotation stages across document batches.
- **Pattern Matching:** Language-specific pipeline configuration dispatch.

## GPU / TPU Support
- Stanford CoreNLP neural models accelerated via DJL GPU backend.
- ONNX-exported Stanza neural models run via ONNX Runtime GPU in Java.
- CoreNLP server mode (REST) allows Java clients to offload to GPU-enabled server.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Stanza wraps Stanford CoreNLP's Java core — use CoreNLP directly.
- Replace `stanza.Pipeline(lang='en')` with `new StanfordCoreNLP(props)`.
- Replace `doc.sentences[0].words` with `coreDoc.sentences().get(0).tokens()`.
- Replace `doc.sentences[0].dependencies` with `GrammaticalStructure` API.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Stanza (StanfordNLP)):**
```python
import stanza
nlp = stanza.Pipeline(lang='en', processors='tokenize,pos,ner,depparse')
doc = nlp('Barack Obama was born in Hawaii.')
for sent in doc.sentences:
    for word in sent.words:
        print(word.text, word.pos, word.deprel)
```

**Java 26 (JStanza):**
```java
import edu.stanford.nlp.pipeline.*;
Properties props = new Properties();
props.setProperty("annotators", "tokenize,ssplit,pos,ner,depparse");
StanfordCoreNLP pipeline = new StanfordCoreNLP(props);
CoreDocument doc = new CoreDocument("Barack Obama was born in Hawaii.");
pipeline.annotate(doc);
for (CoreLabel token : doc.tokens())
    System.out.printf("%s %s%n", token.word(), token.tag());
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>edu.stanford.nlp</groupId>
    <artifactId>stanford-corenlp</artifactId>
    <version>4.5.7</version>
</dependency>
```
