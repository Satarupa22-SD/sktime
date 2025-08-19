# Parallelization and Multithreading in sktime

## Key Concepts

### Multithreading
- Multiple threads of execution within a single process.
- Threads share memory and resources.
- Provides **concurrency** (tasks appear to run at the same time, but may not actually do so simultaneously).
- In Python, the Global Interpreter Lock (GIL) often prevents true parallel execution of Python code with threads.

### Parallelization
- Multiple tasks are executed **simultaneously** (true parallelism).
- Achieved by using multiple CPU cores or processors.
- In Python, this is typically done with **multiprocessing** (multiple processes), especially for CPU-bound tasks.

### In sktime
- When you set `n_jobs` via `set_config`, you enable parallelization for compatible estimators.
- sktime uses the [joblib](https://joblib.readthedocs.io/en/latest/) library, which defaults to **multiprocessing** for CPU-bound tasks.
- This allows true parallel execution on multiple CPU cores.

## Summary Table

| Term            | What it means in sktime context         | How it's achieved         |
|-----------------|----------------------------------------|--------------------------|
| Multithreading  | Multiple threads in one process         | Rarely used for CPU-bound tasks in Python due to GIL |
| Parallelization | Multiple tasks at the same time         | Usually via multiprocessing (joblib, n_jobs > 1)     |
| Concurrency     | Tasks in progress, may be interleaved   | Can be via threads or async, not always parallel      |

## How to Enable Parallelization in sktime

**Old way (deprecated):**
```python
# Deprecated! Do not use
c22 = Catch22(n_jobs=4)
```

**New way (after PR #6002):**
Set the parallelization backend and number of jobs **globally** for all compatible estimators:

```python
from sktime import set_config
set_config(backend="joblib", backend_params={"n_jobs": 4})
```
- `backend="joblib"`: Uses the joblib library for parallelization.
- `n_jobs=4`: Use 4 CPU cores. Set to `-1` to use all available cores.

## Practical Example: Parallel Feature Extraction with Catch22

```python
from sktime import set_config
from sktime.datasets import load_basic_motions
from sktime.transformations.panel.catch22 import Catch22

# Enable parallelization
set_config(backend="joblib", backend_params={"n_jobs": 4})

# Load data
X, y = load_basic_motions(return_X_y=True)

# Feature extraction (this will use 4 processes/cores)
c22 = Catch22()
Xt = c22.fit_transform(X)
print(Xt.head())
```

## Notes
- Not all estimators in sktime support parallelization. Check the documentation for each estimator.
- You can monitor your CPU usage to see parallelization in action.
- To disable parallelization, set `n_jobs=1` in `set_config`.

## In Summary
- Setting `n_jobs` in sktime enables parallelization, typically via multiprocessing, not multithreading.
- This allows for faster computation on multi-core CPUs for supported estimators and transformers.
- Multithreading is not used for CPU-bound tasks in sktime due to Python's GIL. 