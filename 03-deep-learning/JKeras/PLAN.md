# JKeras — Migration Plan: Python Keras → Java 26

## Overview
Keras is the high-level neural network API bundled with TensorFlow. It provides a simple, modular interface for building and training deep learning models with Sequential and Functional APIs.

## Java 26 Equivalent
**DL4J Keras model import** and **DJL Keras-style API**.

## Why Java 26
- **Virtual Threads:** Async data augmentation pipelines feeding training.
- **Vector API:** SIMD-accelerate activation function computation (ReLU, sigmoid).
- **Pattern Matching:** Layer type dispatch in model construction.
- **Structured Concurrency:** Parallel metric computation during validation.

## GPU / TPU Support
- DL4J can import Keras H5 models and run them on CUDA GPU backend.
- DJL loads Keras SavedModel format via TF engine with GPU support.
- ONNX export from Keras → ONNX Runtime Java GPU inference.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Export Keras model: `model.save('model.h5')` or `model.save('saved_model/')`.
- Import in DL4J: `KerasModelImport.importKerasSequentialModelAndWeights(path)`.
- Or load via DJL TF engine: `Criteria.builder().optModelPath(savedModelPath)`.
- Replace Keras callbacks with DL4J `TrainingListener` implementations.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Keras):**
```python
import tensorflow as tf
model = tf.keras.Sequential([
    tf.keras.layers.Dense(128, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dense(10, activation='softmax')
])
model.compile(optimizer='adam', loss='sparse_categorical_crossentropy', metrics=['accuracy'])
model.fit(x_train, y_train, epochs=10)
```

**Java 26 (JKeras):**
```java
import org.deeplearning4j.nn.modelimport.keras.KerasModelImport;
import org.deeplearning4j.nn.multilayer.MultiLayerNetwork;
MultiLayerNetwork model = KerasModelImport
    .importKerasSequentialModelAndWeights("model.h5", true);
model.init();
// Training with DL4J trainer
model.fit(trainDataSetIterator);
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
    <artifactId>deeplearning4j-modelimport</artifactId>
    <version>1.0.0-M2.1</version>
</dependency>
```
