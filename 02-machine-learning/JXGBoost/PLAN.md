# JXGBoost — Migration Plan: Python XGBoost → Java 26

## Overview
XGBoost is the most widely used gradient boosting library. It dominates Kaggle competitions and tabular ML with regularized gradient boosting, built-in cross-validation, and CUDA GPU trees.

## Java 26 Equivalent
**XGBoost4J** (official Java binding) and **H2O GBM**.

## Why Java 26
- **Virtual Threads:** Parallelize tree building across feature splits.
- **Vector API:** SIMD-accelerate histogram bin computation for split finding.
- **Panama FFM:** Direct binding to XGBoost's C++ core via FFM, bypassing JNI.
- **Structured Concurrency:** Parallel boosting round evaluation.

## GPU / TPU Support
- XGBoost4J supports `gpu_hist` tree method — pass `device=cuda` in params.
- H2O GBM supports multi-GPU training on NVIDIA GPUs.
- RAPIDS cuML GPU XGBoost callable from Java via REST or subprocess.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `xgb.DMatrix` with `DMatrix` from `ml.dmlc.xgboost4j`.
- Replace `xgb.train(params, dtrain)` with `XGBoost.train(params, dtrain, ...)`.
- Map param dict keys 1:1 — XGBoost4J uses identical parameter names.
- Replace `model.predict(dtest)` with `booster.predict(dtest)`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (XGBoost):**
```python
import xgboost as xgb
dtrain = xgb.DMatrix(X_train, label=y_train)
params = {'max_depth':6,'eta':0.1,'objective':'binary:logistic'}
model = xgb.train(params, dtrain, num_boost_round=100)
preds = model.predict(xgb.DMatrix(X_test))
```

**Java 26 (JXGBoost):**
```java
import ml.dmlc.xgboost4j.java.*;
DMatrix dtrain = new DMatrix(X_train, nrows, ncols);
dtrain.setLabel(y_train);
Map<String,Object> params = Map.of("max_depth",6,"eta",0.1f,"objective","binary:logistic");
Map<String,DMatrix> watches = Map.of("train",dtrain);
Booster model = XGBoost.train(dtrain, params, 100, watches, null, null);
float[][] preds = model.predict(new DMatrix(X_test, ntest, ncols));
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ml.dmlc</groupId>
    <artifactId>xgboost4j</artifactId>
    <version>2.1.1</version>
</dependency>
```
