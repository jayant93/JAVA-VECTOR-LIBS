# JTriton — Migration Plan: Python Triton Inference Server → Java 26

## Overview
NVIDIA Triton Inference Server is a production model serving platform supporting TensorFlow, PyTorch, ONNX, TensorRT, and custom backends with dynamic batching and model ensemble.

## Java 26 Equivalent
**Triton Java Client** (NVIDIA Triton gRPC/REST client).

## Why Java 26
- **Virtual Threads:** Handle thousands of concurrent inference requests.
- **Structured Concurrency:** Batch request aggregation with timeout.
- **Panama FFM:** Zero-copy tensor transfer between Java and native model runtime.
- **Pattern Matching:** Request type dispatch (REST vs gRPC).

## GPU / TPU Support
ONNX Runtime GPU, TensorRT, and CUDA execution providers available in Java. DJL routes to CUDA-enabled PyTorch/TF engines. Triton Inference Server handles GPU batching server-side.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Set up model loading in application startup with GPU backend configuration.
- Replace Python inference call with Java ONNX Runtime or DJL predictor.
- Implement REST endpoint with Spring Boot or Quarkus.
- Add dynamic batching for throughput optimization.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Triton Inference Server):**
```python
import tritonclient.http as httpclient
client = httpclient.InferenceServerClient('localhost:8000')
input = httpclient.InferInput('input', [1,3,224,224], 'FP32')
input.set_data_from_numpy(data)
result = client.infer('model', [input])
print(result.as_numpy('output'))
```

**Java 26 (JTriton):**
```java
// Triton Java client via gRPC (use grpc-java with Triton protobuf)
import inference.GRPCInferenceServiceGrpc;
import inference.GrpcService.*;
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8001).usePlaintext().build();
var stub = GRPCInferenceServiceGrpc.newBlockingStub(channel);
ModelInferRequest request = ModelInferRequest.newBuilder()
    .setModelName("model")
    .addInputs(InferInputTensor.newBuilder().setName("input")
        .addShape(1).addShape(3).addShape(224).addShape(224).setDatatype("FP32"))
    .build();
ModelInferResponse response = stub.modelInfer(request);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty-shaded</artifactId>
    <version>1.68.1</version>
</dependency>
```
