# JInstructOR — Migration Plan: Python InstructOR → Java 26

## Overview
InstructOR is an instruction-tuned text embedding model that generates task-specific embeddings by prepending task instructions. It achieves state-of-the-art on MTEB and supports few-shot generalization.

## Java 26 Equivalent
**DJL** with ONNX-exported InstructOR embedding model.

## Why Java 26
- **Panama FFM:** Direct ONNX Runtime C API binding for zero-JNI embedding inference.
- **Virtual Threads:** Concurrent instruction-prefixed encoding across document batches.
- **Vector API:** SIMD cosine similarity for retrieval ranking post-processing.
- **Structured Concurrency:** Parallel encode calls with deadline-based cancellation.

## GPU / TPU Support
- ONNX-exported InstructOR runs on CUDA GPU via ONNX Runtime GPU Java.
- TorchScript export and DJL PyTorch engine with CUDA for GPU inference.
- Multi-GPU load balancing via DJL serving predictor pool.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export to ONNX: `torch.onnx.export(model, sample_inputs, 'instructor.onnx')`.
- Load in Java: `OrtSession` with GPU CUDA provider.
- Prepend task instruction to input: `instruction + ': ' + text` before tokenization.
- Use HuggingFace tokenizer for T5-based tokenization (`google/flan-t5-xl` vocab).

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (InstructOR):**
```python
from InstructorEmbedding import INSTRUCTOR
model = INSTRUCTOR('hkunlp/instructor-large')
instruction = 'Represent the Science sentence for retrieval:'
embedding = model.encode([[instruction, 'Quantum physics research.']])
print(embedding.shape) # (1, 768)
```

**Java 26 (JInstructOR):**
```java
import com.microsoft.onnxruntime.*;
OrtEnvironment env = OrtEnvironment.getEnvironment();
OrtSession session = env.createSession("instructor_large.onnx",
    new OrtSession.SessionOptions().addCUDA(0));
// Tokenize: instruction + text with T5 tokenizer
long[][] inputIds = tokenize("Represent the Science sentence for retrieval: Quantum physics.");
OnnxTensor tensor = OnnxTensor.createTensor(env, inputIds);
OrtSession.Result result = session.run(Map.of("input_ids", tensor));
float[][] embedding = (float[][]) result.get("last_hidden_state").getValue();
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>com.microsoft.onnxruntime</groupId>
    <artifactId>onnxruntime_gpu</artifactId>
    <version>1.19.2</version>
</dependency>
```
