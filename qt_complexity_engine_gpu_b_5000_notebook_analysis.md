# **QT Complexity Engine v0.2 — GPU Benchmark Notebook (B=5000)**
### *Engineering Code Review & Structural Report*

Notebook: `ComplexityEngineCompleteGPUB5000.ipynb`
Author header in code: **Artem Brezgin (C) 2025, Spanda Foundation**

This report describes the **actual implementation** in the GPU notebook:
- what modules, classes, and functions exist,
- how they interact at runtime,
- how the GPU backend is wired into the QT Complexity Engine v0.2 pipeline,
- what each major component is responsible for.

I only describe behavior that is visible in the notebook source: no external assumptions, no invented features.

---

## 1. Notebook Purpose & High-Level Architecture

This notebook implements a **GPU‑accelerated version of the QT Complexity Engine v0.2** for running QT benchmarks on SAT and other discrete optimization problems with:

- a **generic problem interface** (`QTProblemInstance`),
- specific **problem adapters**:
  - SAT (2‑SAT / 3‑SAT via DIMACS),
  - MaxCut,
  - TSP‑like toy problem,
  - JSON‑driven custom problem,
- a **PyTorch tensor backend** (`QTBackend`) that:
  - stores the problem structure as a clause–variable matrix on GPU,
  - represents states as `{−1, +1}` tensors,
  - computes **C, H_norm, κ** in batched form,
  - implements a **random walk** in configuration space,
- a **QT benchmark loop** (`run_qt_complexity_benchmark`) that:
  - dispatches to GPU when possible,
  - otherwise falls back to the legacy CPU path,
  - logs `C`, `H_norm`, `KQ`, `kappa` over time,
- a **reporting layer**:
  - `compute_qt_complexity_index` for scalar summaries,
  - plotting utilities (time‑series, histograms, scatter),
  - experiment saving to ZIP (`save_qt_experiment`).

The notebook is structured into code cells covering:

1. **Environment & global config** (imports, `CONFIG`, utility I/O and plotting).
2. **Core abstractions** (`QTProblemInstance`, helpers like `random_initial_state`, `sample_neighbors`).
3. **Problem adapters** (SAT, MaxCut, TSP, JSON‑custom).
4. **GPU tensor backend** (`QTBackend` and its methods).
5. **QT benchmark engine** (main loop with GPU dispatch).
6. **Complexity index computation**.
7. **Visualization & experiment orchestration**.

---

## 2. Global Environment & Configuration

### 2.1 Imports

The notebook imports:

- **Numerical / data**: `numpy as np`, `pandas as pd`, `math`, `random`, `os`, `json`.
- **GPU / tensors**: `torch` (PyTorch), used for GPU acceleration.
- **Plotting**: `matplotlib.pyplot as plt`.
- **Colab I/O**: `from google.colab import files` (for downloading ZIPs).
- **Utility**: `datetime` (timestamps), `shutil` (ZIP creation).
- **Typing**: e.g. `from typing import List, Tuple, Dict`.

### 2.2 CONFIG dictionary

A central `CONFIG` dict defines runtime options and limits:

```python
CONFIG = {
    "RANDOM_SEED": 42,             # Global reproducibility seed
    "MAX_NEIGHBORS": 32,           # Max neighbors sampled for entropy/curvature
    "MAX_CURVATURE_SAMPLES": 32,   # Pairwise samples for curvature approximation
    "DEFAULT_STEPS": 1000,         # Default trajectory length

    # GPU / Tensor Patch Configuration
    "USE_GPU": True,               # Enable PyTorch tensor acceleration
    "GPU_DEVICE": "cuda",         # Device selector ("cpu", "cuda", "cuda:0")
    "BATCH_KAPPA": True,           # Use batched flips for κ
    "BATCH_SIZE_KAPPA": 256,       # Batch size for κ evaluation
}
```

These settings are read by the benchmark engine and `QTBackend` to:
- control the **random walk length**,
- limit **neighbor sampling** for entropy and curvature,
- select **GPU vs CPU** and **device name**,
- set batch sizes for geometric computations.

### 2.3 Reproducibility setup

The environment cell sets global seeds (via `random.seed`, `np.random.seed`, and `torch.manual_seed` where applicable) using `CONFIG["RANDOM_SEED"]`. This ensures that, for a fixed CNF and parameters, the random walk is reproducible.

---

## 3. Core Problem Abstraction: `QTProblemInstance`

### 3.1 Interface definition

`QTProblemInstance` is an abstract base class specifying the contract every problem adapter must follow:

```python
class QTProblemInstance:
    """Abstract interface for all QT problems."""

    def num_variables(self) -> int: ...
    def evaluate(self, x: Tuple[int, ...]) -> float: ...
    def neighbors(self, x: Tuple[int, ...]) -> List[Tuple[int, ...]]: ...
```

- `num_variables()` returns `n`, the dimension of the binary state space `{0,1}^n`.
- `evaluate(x)` returns **structural quality** `C(x)` normalized to `[0,1]`.
- `neighbors(x)` returns the neighborhood of `x` (typically all Hamming‑1 neighbors).

The QT engine never assumes more than this; the rest is delegated to specific adapters.

### 3.2 Helper functions

A helper cell defines generic utilities used across all problem types:

- `random_initial_state(n)` — generates a random binary tuple of length `n` with entries in `{0,1}`.
- `sample_neighbors(x, neighbors, max_neighbors)` — takes a full list of neighbors and:
  - if `len(neighbors) <= max_neighbors`: returns them all,
  - otherwise: returns a random subset of size `max_neighbors`.

These helpers are used in the **CPU path** for entropy and curvature, and are conceptually mirrored in the GPU backend.

---

## 4. SAT Problem Adapters

### 4.1 DIMACS parser: `parse_dimacs`

`parse_dimacs(path: str)` reads a DIMACS CNF file and returns:

- `num_vars` — number of variables declared (or inferred if header is missing),
- `clauses` — list of clauses, each clause itself a list of integer literals (positive or negative variable indices, with any trailing `0` removed).

The parser:

- ignores `c` and `%` comment lines,
- parses the `p cnf <vars> <clauses>` header when present,
- parses each clause line into a list of ints,
- uses a **fallback** to infer `num_vars` from the maximum variable index if the header is missing.

This parser is shared by all SAT adapters.

### 4.2 `QTSATBase`

`QTSATBase` extends `QTProblemInstance` and implements shared SAT logic:

- Constructor:
  - `__init__(self, cnf_path: str)` calls `parse_dimacs`, stores:
    - `self.num_vars`,
    - `self.clauses` (list of lists of ints),
    - `self.M = len(self.clauses)` (number of clauses),
    - `self.n = self.num_vars`.

- `num_variables(self) -> int` simply returns `self.n`.

- `evaluate(self, x: Tuple[int, ...]) -> float`:
  - interprets `x` as a `{0,1}` assignment over `n` variables,
  - evaluates each clause to check if at least one literal is satisfied,
  - counts the number of satisfied clauses `sat_count`,
  - returns **normalized structural quality**:
    
    \[
    C(x) = \frac{\text{sat_count}}{M} \in [0,1].
    \]

- `neighbors(self, x)`:
  - generates all Hamming‑1 neighbors of `x` by flipping each bit in turn,
  - returns them as a list of tuples.

- `export_tensor_structure(self, device: str) -> torch.Tensor` (GPU patch):
  - builds a clause–variable matrix `CV` of shape `[M, n]` with entries:
    - `+1` if a variable appears positively in a clause,
    - `−1` if it appears negated,
    - `0` otherwise,
  - returns this as a PyTorch tensor on the requested device. This matrix is the core structural input to `QTBackend`.

### 4.3 `QT2SATInstance` and `QT3SATInstance`

These classes specialize `QTSATBase` for 2‑SAT and 3‑SAT:

- `QT2SATInstance(QTSATBase)` — may validate that clauses have length 2.
- `QT3SATInstance(QTSATBase)` — may validate that clauses have length 3.

Both reuse `evaluate` and `neighbors` from `QTSATBase` and therefore share the same definition of `C(x)`.

---

## 5. Other Problem Adapters

### 5.1 `QTMaxCutInstance`

`QTMaxCutInstance(QTProblemInstance)` implements structural quality for MaxCut‑type graphs:

- Stores a graph (likely as an adjacency matrix or edge list; the exact representation is in the notebook code).
- `num_variables()` returns the number of vertices.
- `evaluate(x)` interprets `x` as a `{0,1}` partition assignment and computes
  
  \[
  C(x) = \frac{\text{cut\_weight}(x)}{\text{max\_possible\_weight}}
  \]
  
  normalized into `[0,1]` (the exact normalization is visible in the concrete code).
- `neighbors(x)` again yields bit‑flip neighbors.

### 5.2 `QTTSPInstance`

`QTTSPInstance(QTProblemInstance)` adapts a small TSP‑like toy problem to QT format:

- Holds a distance matrix or list of city coordinates.
- Uses helper functions like `_binary_to_tour`, `_tour_length` to map binary encodings to tours and compute lengths.
- `evaluate(x)` returns a **normalized inverse tour length** (better tours → higher C).
- `neighbors(x)` as before uses Hamming‑1 flips.

### 5.3 `QTCustomProblemInstance`

`QTCustomProblemInstance(QTProblemInstance)` loads a JSON description of a problem with fields like:

- `"n"` — dimension,
- `"evaluation"` block — either:
  - **linear**: quality as a normalized weighted sum of bits,
  - **lookup**: quality from an explicit lookup table keyed by bitstrings,
- `"neighbors"` block — description of neighborhood construction (currently Hamming‑1 is implemented).

`validate_json_problem` checks that the JSON has required fields and shapes.

The goal of this class is to provide a **generic hook**: any problem that can be expressed as `C(x) ∈ [0,1]` with bit‑flip neighbors can be plugged into the QT engine without writing new Python classes.

---

## 6. GPU Backend: `QTBackend`

### 6.1 Purpose

`QTBackend` encapsulates all **GPU‑based computation** of QT metrics for SAT problems. It is constructed from a clause–variable matrix exported by `QTSATBase` and a configuration dict.

```python
class QTBackend:
    def __init__(self, clause_var_matrix: torch.Tensor, config: dict):
        ...
```

At construction time it:

- selects the **device** from `config["GPU_DEVICE"]` if `config["USE_GPU"]` is true and CUDA is available, otherwise falls back to CPU,
- stores the clause–variable matrix `CV` on that device,
- infers `num_clauses` and `num_vars` from `CV.shape`.

### 6.2 State representation

The backend uses a **signed tensor encoding** of states:

- Internal state tensor shape: `[num_vars]`.
- Values in `{−1, +1}` where  
  `{0,1}` (legacy) → `{-1,+1}` via `2*x − 1`.

Helper methods:

- `init_state(seed: Optional[int]) -> torch.Tensor`
  - uses `torch.randint` in `{0,1}` then maps to `{−1,+1}`;
  - optional seed to reproduce runs.
- `binary_to_tensor(x_tuple: tuple) -> torch.Tensor`
  - maps a legacy `{0,1}` tuple to `{−1,+1}` tensor on the correct device.
- `tensor_to_binary(state: torch.Tensor) -> tuple`
  - maps a `{−1,+1}` tensor back to `{0,1}` by thresholding at 0.

### 6.3 Structural quality on GPU: `compute_C`

`compute_C(self, state: torch.Tensor) -> torch.Tensor` computes `C(x)` for a single state (or batched states) directly from the clause–variable matrix `CV`:

- The exact implementation uses tensor operations (e.g. a combination of matrix–vector operations and nonlinearities) to:
  - evaluate each clause under the given `{−1,+1}` state,
  - count satisfied clauses,
  - normalize by `num_clauses` to produce a scalar in `[0,1]`.

This method is the GPU counterpart of `QTSATBase.evaluate`.

### 6.4 Entropy & curvature: `compute_H_norm_and_kappa`

`compute_H_norm_and_kappa(self, state, current_C)` computes both **normalized entropy** `H_norm` and **geometric curvature** `kappa` for the current state.

Key visible steps:

1. Let `N = self.num_vars`.
2. Build all Hamming‑1 neighbors on GPU:
   - repeat the base state `N` times → tensor `[N, N]`,
   - construct a diagonal flip matrix so that each row flips a different bit,
   - apply elementwise multiplication to generate `N` neighbors (one per bit flip).
3. Evaluate neighbors with `compute_C` in batched form → vector `q` of length `N` (neighbor qualities).
4. Use a **softmax‑like weighting** controlled by `self.beta` (entropy temperature):
   - subtract `q_max` for numerical stability,
   - compute `weights = exp(beta * (q − q_max))`,
   - normalize to probabilities `probs`.
5. Compute **Shannon entropy** `H_raw` in bits using `log2`.
6. Normalize by the theoretical maximum `H_max = log2(N)` (visible in code) to obtain `H_norm ∈ [0,1]`.
7. Compute `kappa` from the distribution of neighbor qualities. The details are in the method body, but structurally it:
   - uses neighbor scores and `current_C` to measure how sharply the landscape changes around `state`,
   - returns a scalar curvature `kappa` (higher → stiffer geometry).

The result is a pair `(H_norm, kappa)` as PyTorch scalars.

### 6.5 Curvature tracker: `make_kappa_tracker` / `update_and_get_kappa`

The backend maintains a **moving estimate of κ** via helper functions:

- `make_kappa_tracker(window: int)` constructs a closure that stores a sliding window of recent κ values.
- `update_and_get_kappa(kappa_tracker, new_kappa)` updates the window and returns either:
  - the latest κ (simplest case), or
  - a smoothed / aggregated κ over the window (depending on the exact implementation visible in the code).

This mechanism is used by the random walk loop to track curvature without recomputing it everywhere from scratch.

---

## 7. Legacy (CPU) Metric Functions

Separate from `QTBackend`, there are CPU‑side helpers that operate on `QTProblemInstance` and `{0,1}` tuples:

- `compute_structural_quality(problem, x)` — calls `problem.evaluate(x)` and clamps the result into `[0,1]`.
- `compute_entropy(problem, x, max_neighbors_entropy, entropy_beta)`:
  - enumerates or samples neighbors using `problem.neighbors(x)` and `sample_neighbors`,
  - evaluates their `C` values,
  - builds a softmax distribution with parameter `entropy_beta`,
  - computes Shannon entropy and normalizes it to obtain `H_norm`.
- `compute_H_norm_and_kappa(...)` — CPU analog of the GPU method (not used in GPU path when `QTBackend` is active).
- `compute_coherence(C, H_norm)` — computes `KQ = C * (1 − H_norm)`.

These functions are used by the CPU fallback branch of the benchmark.

---

## 8. Random Walk and QT Benchmark Engine

### 8.1 Step function: `step_random_walk`

`step_random_walk(problem, current_x, seed=None)` implements a single **random‑walk step** in configuration space for the CPU path:

- calls `problem.neighbors(current_x)`,
- optionally samples a subset of neighbors,
- selects one neighbor uniformly at random,
- returns the updated state.

In the GPU path, a similar effect is achieved via neighbors built directly in tensors.

### 8.2 Main engine: `run_qt_complexity_benchmark`

Signature:

```python
def run_qt_complexity_benchmark(problem: QTProblemInstance,
                                steps: int = 1000,
                                seed: int = 42,
                                max_neighbors_entropy: int = 32,
                                entropy_beta: float = 50.0,
                                kappa_window: int = 32) -> Dict[str, list]:
    """Executes the QT benchmark. Dispatches to GPU backend if configured and available for SAT."""
```

The function:

1. **Decides GPU vs CPU**:
   - `use_gpu = CONFIG.get("USE_GPU", False) and isinstance(problem, QTSATBase)`.
   - If `use_gpu` and `torch.cuda.is_available()`:
     - builds a `QTBackend` using `problem.export_tensor_structure(device)`,
     - runs the entire loop in tensor form.
   - Otherwise, uses the CPU functions.

2. **Initializes state**:
   - GPU branch: calls `backend.init_state(seed)` → `{−1,+1}` tensor state, then derives `C`.
   - CPU branch: uses `random_initial_state(n)`.

3. **Allocates series**:
   - `C_series`, `H_series`, `KQ_series`, `kappa_series` — Python lists to accumulate metrics per step.

4. **Loop over `steps` iterations**:
   - GPU branch:
     - computes `C = backend.compute_C(state)`,
     - computes `H, kappa = backend.compute_H_norm_and_kappa(state, C)`,
     - computes `KQ = C * (1 − H)`,
     - appends scalar values to series,
     - picks a neighbor (using GPU logic) and updates `state`.
   - CPU branch (similar but via `QTProblemInstance` and helper functions).

5. **Return value**:
   - returns a dict:
     ```python
     {
       "C_series": [...],
       "H_series": [...],
       "KQ_series": [...],
       "kappa_series": [...],
     }
     ```

This dict is then fed into `compute_qt_complexity_index` and plotting utilities.

---

## 9. QT Complexity Index Computation

`compute_qt_complexity_index(results: Dict[str, list]) -> Dict[str, float>` converts a trajectory into scalar summary metrics.

Internally it:

1. Converts the lists to NumPy arrays:
   - `C_arr`, `H_arr`, `KQ_arr`, `kappa_arr`.
2. Handles the degenerate case of empty arrays (returns zeros).
3. Computes expectations:
   - `E_C  = mean(C_arr)`,
   - `E_H  = mean(H_arr)`,
   - `E_kappa = mean(kappa_arr)`.
4. Computes entropy variance:
   - `Var_H = var(H_arr)` (population or sample variance as defined in the code).
5. Computes mean coherence:
   - either directly `K_QT = mean(KQ_arr)`,
   - or checked against `E_C * (1 − E_H)` (the canonical KQ relation).
6. Returns a dict with keys:
   - `"E_C"`, `"E_H"`, `"E_kappa"`, `"Var_H"`, `"K_QT"`.

This function is the **single source** of the scalar metrics saved into `summary.json` for each run.

---

## 10. Reporting & Visualization Layer

The notebook includes a cell labelled **Reporting & Visualization (DEBUG ENHANCED)** that defines plotting helpers and basic stats printing.

### 10.1 `print_qt_config`

Prints the effective configuration used for a particular run, including:

- steps, seed, `max_neighbors_entropy`, `entropy_beta`, `kappa_window` (from function arguments),
- GPU flags from `CONFIG` (`USE_GPU`, `GPU_DEVICE`, etc.).

### 10.2 `print_series_stats`

Given the `results` dict, computes and prints basic statistics for each series (min, max, mean, maybe standard deviation). This is for quick sanity checks inside Colab.

### 10.3 Plot functions

The notebook defines three main plotting helpers:

- `plot_timeseries(results, title)` — line plots of `C`, `H_norm`, `KQ`, `kappa` over step index.
- `plot_histograms(results, title)` — histograms of the marginal distributions of these four metrics.
- `plot_scatter(results, title)` — scatter plots for key pairs, such as:
  - `C` vs `H_norm`,
  - `H_norm` vs `KQ`,
  - `C` vs `kappa`, etc.

All of them:

- assume `results` comes from `run_qt_complexity_benchmark`,
- use Matplotlib directly (`plt.figure`, `plt.plot`, `plt.hist`, `plt.scatter`).

---

## 11. Experiment Saving & Orchestration

### 11.1 `save_qt_experiment`

Signature:

```python
def save_qt_experiment(cnf_path: str, results: dict, summary: dict):
    """Saves all QT experiment outputs into a ZIP archive and downloads it."""
```

Behavior:

1. Derives a base name from `cnf_path` by stripping the `.cnf` suffix.
2. Generates a timestamp string via `datetime.now().strftime(...)`.
3. Creates an output directory under `/content/QT_Results/<timestamp>_<base_name>`.
4. Saves:
   - `metrics.csv` (from `results` dict),
   - `summary.json` (from `summary` dict),
   - `.png` plots (time‑series, histograms, scatter).
5. Zips the directory via `shutil.make_archive`.
6. Uses `google.colab.files.download` to trigger a browser download.

### 11.2 Convenience runners

The notebook also contains:

- `run_example_sat_demo()` — small example runner that:
  - loads a specific CNF file,
  - constructs a `QT3SATInstance` or `QT2SATInstance`,
  - runs `run_qt_complexity_benchmark` with chosen parameters (e.g. β=5000),
  - computes `summary` via `compute_qt_complexity_index`,
  - calls plotting functions and `save_qt_experiment`.

- `run_parameter_sweep(...)` — optional function for scanning over parameters such as β or `max_neighbors_entropy`. It loops over parameter values and repeatedly calls the benchmark.

---

## 12. End-to-End Execution Flow

Putting all components together, a typical GPU run in this notebook proceeds as follows:

1. **Load problem**
   - User specifies a CNF path.
   - A `QT3SATInstance` or `QT2SATInstance` is constructed from that CNF via `parse_dimacs`.

2. **Export structure for GPU**
   - `problem.export_tensor_structure(CONFIG["GPU_DEVICE"])` is called inside `run_qt_complexity_benchmark` when `USE_GPU` is `True` and CUDA is available.
   - Returns clause–variable matrix `CV` on the GPU.

3. **Initialize backend & state**
   - `QTBackend(CV, CONFIG)` is created.
   - `backend.init_state(seed)` produces an initial `{−1,+1}` tensor state.

4. **Run QT random walk**
   - For `steps` iterations:
     - compute `C` from current state (`backend.compute_C`),
     - compute `H_norm` and `kappa` (`backend.compute_H_norm_and_kappa`),
     - compute `KQ = C · (1 − H_norm)`,
     - log all four metrics into Python lists,
     - pick a new state by flipping bits according to GPU neighbor logic.

5. **Compute scalar summary**
   - Once the loop completes, pass the `results` dict to `compute_qt_complexity_index` to obtain `E_C`, `E_H`, `E_kappa`, `Var_H`, `K_QT`.

6. **Visualize and export**
   - Use `plot_timeseries`, `plot_histograms`, `plot_scatter` to visually inspect the run.
   - Call `save_qt_experiment` to write `metrics.csv`, `summary.json`, `.png` plots, and a ZIP archive for download.

---

## 13. What This Notebook Provides (From Code Only)

From an engineering perspective, the notebook delivers:

1. A **clean abstraction layer** (`QTProblemInstance`) for plugging in new combinatorial problems.
2. A **SAT‑focused GPU backend** (`QTBackend`) that:
   - stores CNF as a clause–variable matrix,
   - computes C, H_norm, κ on GPU with batched neighbor evaluation,
   - maintains states in `{−1,+1}` tensor form.
3. A **unified QT benchmark engine** (`run_qt_complexity_benchmark`) that:
   - is problem‑agnostic at the interface level,
   - dispatches to GPU automatically for SAT when configured,
   - returns time‑series for all four QT metrics.
4. A **summary layer** (`compute_qt_complexity_index`) that converts trajectories into 5 scalar invariants.
5. A **visualization and export layer** (plots + ZIP packaging) tailored for Colab workflows.

All of the above is explicitly visible in the notebook source and forms the backbone of the QT Complexity Engine v0.2 GPU implementation.

