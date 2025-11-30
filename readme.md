# QT Complexity Engine — README.md (Full Repository Specification)

### Author: Artem Brezgin (C) 2025, Spanda Foundation

---

# **0. Overview**

The **QT Complexity Engine** is a computational framework based on the Quant‑Trika (QT) formalism — a meta‑ontology of coherence that models structure, entropy, curvature, and global coherence in complex systems.

This repository contains:

- the **core engine implementation** (Python + CUDA‑ready);
- modules for loading and analyzing SAT, MaxCut, and other constraint‑based instances;
- a **canonical evaluator** of QT metrics: `C`, `H_norm`, `kappa`, `KQ`;
- data pipelines for CSV, JSON, and visualization outputs;
- full **benchmark results** for the QT SAT dataset (5 instances);
- detailed engineering documentation and theoretical background.

The engine is built for both **scientific analysis** and **benchmarking** of computational problems under the Quant‑Trika lens.

---

# **1. What Problems the Engine Solves**

Traditional complexity measures (runtime, clause count, graph size) often fail to describe how a solver experiences a search landscape. The QT Complexity Engine provides a **geometric, entropic, and structural map** of the instance.

It answers:

### **1.1 How structured is the problem?**

Measured by the **Structural Quality** metric `C`, which quantifies the meaningful, persistent order in the constraint graph.

### **1.2 How turbulent is the search space?**

Measured by **Normalized Entropy** `H_norm`, capturing conflict, uncertainty, and instability.

### **1.3 How stiff or curved is the landscape?**

Measured by **Computational Curvature** `kappa`, indicating how constrained the manifold is.

### **1.4 How much global coherence emerges?**

Measured by **KQ**, computed canonically as:

```
KQ = C · (1 − H_norm)
```

This is the core invariant of Quant‑Trika.

### **1.5 What dynamical regime does the instance belong to?**

The engine identifies:

- Regime I+ — ultra‑coherent, low entropy
- Regime I  — low entropy, structured
- Regime II — mid‑entropy oscillatory
- Regime III — high‑entropy but geometrically stable
- Regime IV — entropy collapse

### **1.6 How does the instance behave over time?**

The engine generates:

- time‑series CSVs;
- scatterplots;
- histograms;
- phase‑space trajectories.

Together these form a **complete dynamical portrait** of any computational instance.

---

# **2. Repository Structure**

```
qt-complexity-engine-GPU/
│
├── qt_complexity_engine_gpu_b_5000_notebook_analysis.md   # QT Complexity Engine v0.2 — GPU Benchmark Notebook (B=5000) methods
├── qt_metrics_conceptual_measurement_primer.md            # Quant‑Trika Metric System — Conceptual Primer
├── ComplexityEngineCompleteGPUB5000.ipynb                 # Fully reproduceble Google Colab Complexity Engine for GPU Notebook
├── uuf_250_batch_appendix.md                              # Comparative Analysis of 10 QT Engine Runs uuf250‑091-uuf250‑0100
├── UUF250Familybatch.zip                                  # Full 10 run artifacts for uuf250 family batch
├── uuf_250_090.md                                         # QT Complexity Engine Report — uuf250-090
├── c_880_mul_miter_shuffled_as.md                         # QT Complexity Engine Report — C880mul.miter
├── bart_30.md                                             # QT Complexity Engine Report — bart30
├── am_9_9_shuffled_as.md                                  # QT Complexity Engine Report — am_9_9
├── alu_4_mul_miter_shuffled_as.md                         # QT Complexity Engine Report — alu4mul.miter
├── alu4mul.miter.shuffled-as.sat03-344.zip                # Full run artifacts for alu4mul run
├── C880mul.miter.shuffled-as.sat03-348.zip                # Full run artifacts for C880mul run
├── bart30.shuffled.zip                                    # Full run artifacts for bart30 run
├── am_9_9.shuffled-as.sat03-365.zip                       # Full run artifacts for am_9_9 run
├── uuf250-090.zip                                         # Full run artifacts for uuf250-90 run
├── LICENSE
└── README.md                                              # (this file)
```

---

#

```bash
pip install -r requirements.txt
```

Optional GPU modules:

```bash
pip ins
```

# **4. Quick Start**

### **4.1 Running the Engine on a DIMACS SAT Instance**

```bash
python run.py --input path/to/formula.cnf --steps 1000 --beta 5000
```

Output:

- `metrics.csv`
- `summary.json`
- PNG plots (time‑series, histograms, scatter)

### **4.2 Running All Benchmarks**

```bash
bash scripts/run_all.sh
```

---

# **5. Quant‑Trika Metrics (Core Definitions)**

### **5.1 Structural Quality (C)**

Quantifies coherence‑supporting structural order. Derived from clause connectivity, pattern density, and constraint alignment.

### **5.2 Normalized Entropy (H\_norm)**

Measures local turbulence and instability in the evolving assignment.

### **5.3 Computational Curvature (kappa)**

Captures how rigid or flexible the local constraint manifold is.

### **5.4 Coherence (KQ)**

The central Quant‑Trika invariant:

```
KQ = C · (1 − H_norm)
```

Represents surviving structure under entropy.

---

# **6. Dynamical Regimes**

The engine classifies each step into one of five regimes:

| Regime | Description                          |
| ------ | ------------------------------------ |
| I+     | Ultra‑low entropy, maximal coherence |
| I      | Low entropy, structured stability    |
| II     | Mid‑entropy oscillatory dynamics     |
| III    | High entropy, geometry‑constrained   |
| IV     | Extreme entropy collapse             |

This forms the core of the **QT dynamical atlas** for SAT/MaxCut landscapes.

---

# **7. Benchmark Results (Summary)**

The repository includes full analyses for five canonical SAT instances:

- `alu4mul.miter.shuffled-as.sat03-344`
- `C880mul.miter.shuffled-as.sat03-348`
- `bart30.shuffled`
- `am_9_9.shuffled-as.sat03-365`
- `uuf250-090`

For each instance:

- raw CSV metrics (1000 steps)
- JSON aggregate metrics
- time‑series plots
- scatter matrices
- histograms
- phase-segment analysis
- full PDF/markdown reports

Detailed comparison across all five instances is provided in `benchmarks/README.md`.

---

# **8. Theoretical Documentation**

Located in `docs/theory/`:

### **8.1 QT Foundations**

Introduces the ontology of structure, entropy, curvature, coherence.

### **8.2 Coherence Law**

Formalizes the KQ equation and its geometric meaning.

### **8.3 QT Metrics**

Defines all invariants mathematically, including edge cases.

---

# **9. Engineering Documentation**

Found in `docs/engine_spec/`.

### Includes:

- full pipeline (loader → engine → analysis → export)
- API documentation
- implementation notes
- reproducibility constraints
- reference experiments

---

# **10. Reproducibility & Data Integrity**

All outputs are deterministic given:

- input seed
- beta schedule
- number of steps

The repository includes:

- seeds for benchmark runs
- versioned CSV/JSON files
- raw logs

---

# **11. Contributing**

Contributions are welcome. Please submit:

- feature proposals
- bug reports
- engine accuracy improvements
- documentation enhancements

---

# **12. License**

This project is released under the **MIT License**.

---

# **13. Citation**

If you use the QT Complexity Engine in research:

```
Brezgin, Artem (2025). QT Complexity Engine — A Quant‑Trika Framework for Structural, Entropic, and Curvature-Based Analysis. Spanda Foundation.
```

---

# **14. Contact**

For research collaboration, consulting, or academic inquiries:
**Artem Brezgin — Spanda Foundation**

Feel free to reach out via GitHub issues or email.

