# JScikitLearn — Migration Plan: Python scikit-learn → Java 26

## Overview
scikit-learn is Python's industry-standard ML library offering classification, regression, clustering, dimensionality reduction, and model selection with a consistent fit/predict API.

## Java 26 Equivalent
**Smile** (Statistical Machine Intelligence and Learning Engine) and **Weka**.

## Why Java 26
- **Virtual Threads:** Parallelize cross-validation folds across lightweight threads.
- **Vector API:** SIMD-accelerate distance computations in KNN and clustering.
- **Pattern Matching:** Clean model pipeline dispatch and hyperparameter handling.
- **Structured Concurrency:** Safe parallel model evaluation across folds.

## GPU / TPU Support
- Smile supports GPU acceleration for select algorithms via ND4J CUDA.
- H2O backend enables distributed GPU training on clusters.
- ONNX export from Smile models → run via ONNX Runtime GPU provider.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Replace `sklearn.linear_model.LogisticRegression` with `smile.classification.LogisticRegression`.
- Replace `Pipeline` with Smile's `Pipeline` or manual chaining.
- Replace `train_test_split` with Smile's `MathEx.permutate` and index slicing.
- Replace `GridSearchCV` with H2O AutoML or manual virtual-thread grid search.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (scikit-learn):**
```python
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
X_train,X_test,y_train,y_test = train_test_split(X,y)
model = LogisticRegression().fit(X_train, y_train)
print(model.score(X_test, y_test))
```

**Java 26 (JScikitLearn):**
```java
import smile.classification.LogisticRegression;
import smile.validation.CrossValidation;
double[][] X = ...; int[] y = ...;
LogisticRegression model = LogisticRegression.fit(X, y);
double accuracy = CrossValidation.classification(5, X, y,
    (tx, ty) -> LogisticRegression.fit(tx, ty)).avg.accuracy;
System.out.println(accuracy);
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>com.github.haifengl</groupId>
    <artifactId>smile-core</artifactId>
    <version>3.1.1</version>
</dependency>
```
