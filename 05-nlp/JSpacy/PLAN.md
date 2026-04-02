# JSpacy — Migration Plan: Python spaCy → Java 26

## Overview
spaCy is an industrial-strength NLP library with fast tokenization, POS tagging, dependency parsing, NER, and word vectors. It supports neural models (transformers) via spaCy-transformers.

## Java 26 Equivalent
**Stanford CoreNLP** for industrial-strength NLP pipeline in Java.

## Why Java 26
- **Virtual Threads:** Process thousands of documents concurrently.
- **Vector API:** SIMD-accelerate word vector similarity computation.
- **Panama FFM:** Direct binding to Stanford CoreNLP C++-native components.
- **Structured Concurrency:** Pipeline stage parallelism (tokenize → parse → NER).

## GPU / TPU Support
- Stanford CoreNLP neural models run on GPU via DJL integration.
- ONNX-exported spaCy models (`spacy-onnx`) run via ONNX Runtime GPU in Java.
- GPU-accelerated NER via DJL transformer models as CoreNLP replacement.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Start `StanfordCoreNLP` pipeline with annotators: `tokenize, ssplit, pos, lemma, ner, parse`.
- Replace `doc = nlp(text)` with `pipeline.process(text)` → `CoreDocument`.
- Replace `doc.ents` with `doc.entityMentions()`.
- Replace `token.dep_` with `CoreLabel.get(EnhancedDependenciesAnnotation.class)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (spaCy):**
```python
import spacy
nlp = spacy.load('en_core_web_sm')
doc = nlp('Apple is looking at buying U.K. startup for $1 billion')
for ent in doc.ents:
    print(ent.text, ent.label_)
```

**Java 26 (JSpacy):**
```java
import edu.stanford.nlp.pipeline.*;
import edu.stanford.nlp.ling.*;
Properties props = new Properties();
props.setProperty("annotators", "tokenize,ssplit,pos,lemma,ner");
StanfordCoreNLP pipeline = new StanfordCoreNLP(props);
CoreDocument doc = new CoreDocument("Apple is looking at buying U.K. startup.");
pipeline.annotate(doc);
for (CoreEntityMention em : doc.entityMentions())
    System.out.println(em.text() + " -> " + em.entityType());
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
