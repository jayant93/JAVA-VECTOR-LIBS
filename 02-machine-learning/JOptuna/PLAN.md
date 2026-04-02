# JOptuna — Migration Plan: Python Optuna → Java 26

## Overview
Optuna is a hyperparameter optimization framework using Bayesian optimization with Tree-structured Parzen Estimators (TPE), pruning via Hyperband, and distributed parallelism.

## Java 26 Equivalent
**H2O AutoML** and manual Bayesian optimization with **Apache Commons Math**.

## Why Java 26
- **Virtual Threads:** Run hundreds of parallel trials as lightweight threads.
- **Structured Concurrency:** Fan out trial evaluations, collect best result, cancel the rest.
- **Vector API:** Accelerate surrogate model (Gaussian Process) computation.
- **Pattern Matching:** Clean trial parameter type dispatch.

## GPU / TPU Support
- H2O AutoML uses multi-GPU gradient boosting internally.
- ONNX Runtime GPU for fast trial model evaluation during search.
- Optuna's Python server accessible from Java via REST for remote study management.

## Migration Phases

### Phase 1 – Setup
- Add required Maven dependencies and configure the Java 26 project.
- Set up backend selection (CPU/GPU) via configuration or environment variables.
- Establish logging and resource management.

### Phase 2 – Core Migration
- Use H2O AutoML for automated model selection and tuning.
- For custom search: implement TPE or random search with virtual threads.
- Replace `optuna.create_study()` with H2O `AutoML` builder.
- Replace `study.optimize(objective, n_trials)` with `automl.train()`.

### Phase 3 – Optimization
- Profile hot paths and apply Java Vector API (SIMD) for element-wise operations.
- Use virtual threads for concurrent data loading and preprocessing pipelines.
- Apply off-heap `MemorySegment` (Panama FFM) for zero-copy native interop.

### Phase 4 – GPU/Vector Acceleration
- Enable GPU backend by swapping the Maven dependency (CUDA/OpenCL).
- Use structured concurrency to fan out GPU kernel submissions.
- Benchmark with JMH to validate SIMD and GPU throughput gains.

## API Comparison

**Python (Optuna):**
```python
import optuna
def objective(trial):
    lr = trial.suggest_float('lr', 1e-4, 1e-1, log=True)
    return train_and_eval(lr)
study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=100)
```

**Java 26 (JOptuna):**
```java
import ai.h2o.automl.AutoML;
import ai.h2o.automl.AutoMLBuildSpec;
H2O.init();
AutoMLBuildSpec spec = new AutoMLBuildSpec();
spec.input_spec.training_frame = trainFrame;
spec.input_spec.response_column = "label";
spec.build_control.max_models = 100;
AutoML automl = AutoML.startAutoML(spec);
automl.get();
System.out.println(automl.leaderboard());
```

## Performance Considerations
- Java 26 Vector API provides SIMD-width operations (AVX-512 / ARM NEON) for numerical kernels.
- Foreign Function & Memory API enables zero-copy data exchange with native C/C++ libraries.
- Virtual Threads (Project Loom) allow thousands of concurrent I/O-bound tasks without blocking.
- Structured Concurrency ensures clean cancellation and error propagation across parallel tasks.

## Dependencies (Maven)
```xml
<dependency>
    <groupId>ai.h2o</groupId>
    <artifactId>h2o-automl</artifactId>
    <version>3.46.0.6</version>
</dependency>
```
