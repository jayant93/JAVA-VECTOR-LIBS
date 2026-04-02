# JCatBoost — Migration Plan: Python CatBoost → Java 26

## Overview
CatBoost is Yandex's gradient boosting library with native categorical feature support, ordered boosting to prevent target leakage, and strong out-of-the-box performance without extensive preprocessing.

## Java 26 Equivalent
**CatBoost4J** (official JVM library from Yandex).

## Why Java 26
- **Virtual Threads:** Parallel fold evaluation during cross-validation.
- **Vector API:** SIMD dot-product acceleration for leaf value computation.
- **Panama FFM:** Native CatBoost C++ library access without JNI boilerplate.
- **Pattern Matching:** Feature type dispatch (numerical vs. categorical).

## GPU / TPU Support
- CatBoost4J supports GPU training via `task_type=GPU` parameter.
- Multi-GPU training supported with `devices` parameter.
- ONNX export from CatBoost → run via ONNX Runtime GPU provider in Java.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Add `catboost4j-prediction` Maven dependency.
- Replace `CatBoostClassifier().fit(X, y, cat_features)` with `CatBoostModel.loadModel()`.
- For training, use CatBoost CLI and load the resulting model in Java.
- Replace `model.predict_proba(X)` with `model.predict(features)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (CatBoost):**
```python
from catboost import CatBoostClassifier
model = CatBoostClassifier(iterations=100, depth=6, learning_rate=0.1)
model.fit(X_train, y_train, cat_features=['cat_col'])
preds = model.predict_proba(X_test)
```

**Java 26 (JCatBoost):**
```java
import ai.catboost.CatBoostModel;
import ai.catboost.CatBoostPredictions;
CatBoostModel model = CatBoostModel.loadModel("model.cbm");
// Prepare float[][] numericFeatures and String[][] catFeatures
CatBoostPredictions result = model.predict(numericFeatures, catFeatures);
System.out.println(result.get(0, 0)); // probability for first sample
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ai.catboost</groupId>
    <artifactId>catboost4j-prediction</artifactId>
    <version>1.2.7</version>
</dependency>
```
