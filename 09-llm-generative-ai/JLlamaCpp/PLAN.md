# JLlamaCpp — Migration Plan: Python llama-cpp-python → Java 26

## Overview
llama-cpp-python provides Python bindings for llama.cpp — a C++ LLM inference engine supporting quantized (GGUF) models with CPU, CUDA, Metal, and OpenCL backends for local inference.

## Java 26 Equivalent
**LLaMA Java bindings** (llama.cpp JNI/Panama FFM wrapper).

## Why Java 26
- **Panama FFM (JEP 454):** Direct zero-JNI binding to llama.cpp C API for model loading and generation.
- **Virtual Threads:** Async token generation without blocking the carrier thread pool.
- **Vector API:** SIMD-accelerate CPU-side token sampling and logit processing.
- **Structured Concurrency:** Parallel prefill and decode phase management.

## GPU / TPU Support
llama.cpp natively supports CUDA (NVIDIA), ROCm (AMD), and Metal (Apple). Java binds via Panama FFM to llama.cpp shared library — same GPU backends available. GGUF quantized models (Q4_K_M, Q8_0) maximize GPU memory efficiency.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Build llama.cpp as a shared library (`cmake -DLLAMA_CUDA=ON -DBUILD_SHARED_LIBS=ON`).
- Bind `llama_init_from_model`, `llama_decode`, `llama_sample_token` via Panama FFM `SymbolLookup`.
- Replace `llm(prompt, max_tokens)` with Panama FFM native call sequence.
- Replace Python iterator-based streaming with Java virtual thread + blocking queue pattern.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (llama-cpp-python):**
```python
from llama_cpp import Llama
llm = Llama(model_path='llama-3.1-8b-q4_k_m.gguf', n_gpu_layers=-1)
output = llm('Q: What is Java 26? A:', max_tokens=256, stop=['Q:'])
print(output['choices'][0]['text'])
```

**Java 26 (JLlamaCpp):**
```java
// Panama FFM binding to llama.cpp
import java.lang.foreign.*;
import java.lang.invoke.MethodHandle;
SymbolLookup llamaLib = SymbolLookup.libraryLookup("libllama.so", Arena.global());
MethodHandle llamaInit = Linker.nativeLinker().downcallHandle(
    llamaLib.find("llama_init_from_model").orElseThrow(),
    FunctionDescriptor.of(ValueLayout.ADDRESS, ValueLayout.ADDRESS, ValueLayout.ADDRESS));
// Full inference pipeline via Panama FFM calls...
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<!-- No Maven artifact; build llama.cpp as shared lib and use Panama FFM -->
<!-- Or use LangChain4j Ollama which wraps llama.cpp via Ollama server -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-ollama</artifactId>
    <version>0.35.0</version>
</dependency>
```
