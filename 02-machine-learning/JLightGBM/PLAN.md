# JLightGBM — Migration Plan: Python LightGBM → Java 26

## Overview
LightGBM is Microsoft's fast gradient boosting library using leaf-wise tree growth and histogram-based splitting. It uses less memory than XGBoost and trains faster on large datasets.

## Java 26 Equivalent
**LightGBM4j** (Java binding) and **H2O GBM**.

## Why Java 26
- **Virtual Threads:** Concurrent feature histogram building across partitions.
- **Vector API:** SIMD histogram accumulation for split gain computation.
- **Panama FFM:** Zero-JNI-overhead bindings to LightGBM C API.
- **Structured Concurrency:** Parallel bagging rounds execution.

## GPU / TPU Support
- LightGBM4j supports GPU tree learner via `device_type=gpu` parameter.
- H2O GBM runs on multi-GPU clusters via H2O Sparkling Water.
- ONNX export: LightGBM → ONNX → Java ONNX Runtime GPU inference.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Use `io.github.metarank/lightgbm4j` Java binding.
- Replace `lgb.Dataset` with `LGBMDataset.createFromMat()`.
- Replace `lgb.train(params, train_data)` with `LGBMBooster.create(dataset, params)`.
- Replace `model.predict(X)` with `booster.predictForMat(X, ...)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (LightGBM):**
```python
import lightgbm as lgb
train_data = lgb.Dataset(X_train, label=y_train)
params = {'num_leaves':31,'learning_rate':0.05,'n_estimators':100}
model = lgb.train(params, train_data)
preds = model.predict(X_test)
```

**Java 26 (JLightGBM):**
```java
import io.github.metarank.lightgbm4j.*;
LGBMDataset dataset = LGBMDataset.createFromMat(X_train, nrows, ncols, true, "", null);
dataset.setField("label", y_train);
LGBMBooster booster = LGBMBooster.create(dataset,
    "num_leaves=31 learning_rate=0.05 n_estimators=100");
for (int i = 0; i < 100; i++) booster.updateOneIter();
double[] preds = booster.predictForMat(X_test, ntest, ncols, true);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>io.github.metarank</groupId>
    <artifactId>lightgbm4j</artifactId>
    <version>4.3.0-1</version>
</dependency>
```
