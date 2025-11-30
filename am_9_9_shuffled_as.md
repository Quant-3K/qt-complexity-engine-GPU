# QT Complexity Engine Report — `am_9_9.shuffled-as.sat03-365`

## 0. Instance & Inputs

**Instance ID:** `am_9_9.shuffled-as.sat03-365`

**Archive contents (for this analysis):**
- `summary.json` — global aggregate metrics.
- `metrics.csv` — time-series for `C`, `H_norm`, `KQ`, `kappa` over 1000 steps.
- `timeseries.png` — time-series plots (used only qualitatively).
- `histograms.png` — histograms of `H_norm` and `kappa`.
- `scatter.png` — scatter plots between selected metrics.

All quantitative statements below are based **directly on `summary.json` and `metrics.csv`**. Images are used only to confirm shapes and support qualitative interpretation.

---

## 1. Sanity & Consistency Checks

From `summary.json`:

- `E_C`      = 0.8392019702792167
- `E_H`      = 0.9940440260171890
- `E_kappa`  = 0.9958960211873055
- `Var_H`    = 0.0000018605850868
- `K_QT`     = 0.004998306818089805
- `E_Beta`   = 5000.0
- `Max_Beta` = 5000.0

From `metrics.csv` (`count = 1000` rows):

- mean(C)      ≈ 0.839202
- mean(H_norm) ≈ 0.994044
- mean(KQ)     ≈ 0.004998
- mean(kappa)  ≈ 0.995896

The CSV means match the JSON aggregates to numerical precision, so **the JSON summary is consistent with the raw time-series data**.

### 1.1 Canonical relation check

We test the canonical Quant-Trika relation

> **KQ ≈ C · (1 − H_norm)**

pointwise over all 1000 steps, defining `KQ_pred = C * (1 − H_norm)` and `residual = KQ − KQ_pred`.

Residual statistics:

- mean ≈ −5.28×10⁻¹⁷
- std  ≈ 6.98×10⁻¹⁷
- min  ≈ −1.92×10⁻¹⁶
- max  ≈ 9.28×10⁻¹⁷

These are at floating-point roundoff scale, so **KQ is numerically identical to C·(1−H_norm)** for this instance. No inconsistencies with the canonical formula.

### 1.2 Range sanity

From `metrics.csv`:

- `C`      ∈ [0.838427, 0.840311]
- `H_norm` ∈ [0.988451, 0.998020]
- `KQ`     ∈ [0.001663, 0.009687]
- `kappa`  ∈ [0.994281, 0.997579]

All metrics lie inside expected physical ranges. There are **no NaNs, infinities, or obvious outliers**.

---

## 2. Global Metrics (JSON view)

Using the `summary.json` values:

- **Structural Quality (E_C) ≈ 0.8392**  
  Structure is **high but slightly lower** than in the SAT instances `alu4mul` and `C880mul` (which had E_C ≈ 0.875–0.878). The configuration space is still strongly organized.

- **Entropy (E_H) ≈ 0.9940, Var_H ≈ 1.86×10⁻⁶**  
  Entropy is **extremely high**, very close to its maximum value, and its variance is **tiny**. This means the system stays almost always in an **ultra-high-entropy state** with only microscopic fluctuations.

- **Curvature (E_kappa) ≈ 0.9959**  
  Curvature is very high, slightly above 0.995 on average, again with small variability. The geometric manifold is **stiff and tightly constrained**.

- **Coherence (K_QT) ≈ 0.0050**  
  Coherence is **almost fully suppressed**. A value around 0.005 relative to C ≈ 0.839 means that meaningful global order is practically annihilated by entropy.

- **Beta control parameter: E_Beta = Max_Beta = 5000**  
  As in the other instances, β is locked at its maximum for the entire run.

**Interpretation:** globally, `am_9_9.shuffled-as.sat03-365` lives in an **extreme high-entropy, near-zero-coherence regime**. Structure and curvature remain high, but they are buried under almost maximal turbulence.

---

## 3. Time-Series Structure (CSV view)

### 3.1 Overall descriptive statistics

Full-run descriptive statistics from `metrics.csv`:

- **C (structural quality)**  
  - mean ≈ 0.839202  
  - std  ≈ 0.000509  
  - min/max ≈ [0.838427, 0.840311]  
  - quantiles:  
    - 10% ≈ 0.83864  
    - 25% ≈ 0.83877  
    - 50% ≈ 0.83903  
    - 75% ≈ 0.83962  
    - 90% ≈ 0.84002  
  
  Structural quality is **very stable**: it fluctuates only at the 4th decimal place. There is no sign of structural breakdown.

- **H_norm (normalized entropy)**  
  - mean ≈ 0.994044  
  - std  ≈ 0.001365  
  - min/max ≈ [0.988451, 0.998020]  
  - quantiles:  
    - 10% ≈ 0.99228  
    - 25% ≈ 0.99315  
    - 50% ≈ 0.99410  
    - 75% ≈ 0.99499  
    - 90% ≈ 0.99578  
  
  H_norm is **tightly concentrated** in [0.99, 0.996], with small Gaussian-like spread. The histogram confirms a single sharp peak near ≈0.994. The system is **locked into a high-entropy plateau**.

- **KQ (coherence)**  
  - mean ≈ 0.004998  
  - std  ≈ 0.001145  
  - min/max ≈ [0.001663, 0.009687]  
  - quantiles:  
    - 10% ≈ 0.00354  
    - 25% ≈ 0.00420  
    - 50% ≈ 0.00495  
    - 75% ≈ 0.00575  
    - 90% ≈ 0.00648  
  
  Coherence is **tiny but not identically zero**, fluctuating within [≈0.0017, ≈0.0097]. This matches the expectation from KQ = C·(1−H_norm) with C ≈ 0.839 and H_norm ≈ 0.994.

- **kappa (curvature)**  
  - mean ≈ 0.995896  
  - std  ≈ 0.000473  
  - min/max ≈ [0.994281, 0.997579]  
  - quantiles:  
    - 10% ≈ 0.99532  
    - 25% ≈ 0.99556  
    - 50% ≈ 0.99589  
    - 75% ≈ 0.99622  
    - 90% ≈ 0.99649  
  
  Curvature is very high and tightly clustered, confirming a **stiff, highly constrained geometry** throughout the run.

### 3.2 Segment-wise statistics

To check for hidden phases or drifts, we split the 1000-step run into three segments:

1. **Segment 1 — steps 0–349 (350 steps)**  
   - mean(C)   ≈ 0.839817  
   - mean(H)   ≈ 0.993961, std(H) ≈ 0.001335  
   - min/max H ≈ [0.989693, 0.998020]  
   - mean(KQ)  ≈ 0.005072, std(KQ) ≈ 0.001121  
   - mean(kappa) ≈ 0.995871, std(kappa) ≈ 0.000455  

2. **Segment 2 — steps 350–749 (400 steps)**  
   - mean(C)   ≈ 0.838877  
   - mean(H)   ≈ 0.994118, std(H) ≈ 0.001349  
   - min/max H ≈ [0.989829, 0.997758]  
   - mean(KQ)  ≈ 0.004934, std(KQ) ≈ 0.001132  
   - mean(kappa) ≈ 0.995921, std(kappa) ≈ 0.000471  

3. **Segment 3 — steps 750–999 (250 steps)**  
   - mean(C)   ≈ 0.838860  
   - mean(H)   ≈ 0.994041, std(H) ≈ 0.001429  
   - min/max H ≈ [0.988451, 0.997011]  
   - mean(KQ)  ≈ 0.004999, std(KQ) ≈ 0.001199  
   - mean(kappa) ≈ 0.995891, std(kappa) ≈ 0.000501  

**Facts:**

- All three segments have **almost identical** entropies (≈0.994) and coherences (≈0.005).  
- Structural quality C is slightly higher in Segment 1, but the difference is small (~0.001).  
- Curvature is consistently high with only tiny drift.

**Interpretation:**

- There is **no meaningful phase structure** in this run: the engine remains in a single, ultra-high-entropy plateau from start to finish.  
- Any apparent drifts in the plots are very small compared to the absolute level of entropy and coherence.

### 3.3 Temporal narrative

- **Fact:**  
  - `H_norm(t)` is almost flat near 0.994 for all t.  
  - `KQ(t)` is almost flat near 0.005 for all t.  
  - `C(t)` and `kappa(t)` show only small random-like wander around their means.  

- **Interpretation:**  
  - The solver **enters an extreme-entropy regime almost immediately and never exits it**.  
  - The landscape explored is a **single turbulent ocean** with almost no pockets of lower entropy or higher coherence.

---

## 4. Regime Classification

Using the qualitative regimes:

- **Regime I+ — Ultra-Low Entropy Coherence**
- **Regime I  — Low-Entropy Coherence**
- **Regime II — Mid-Entropy Oscillation**
- **Regime III — High-Entropy Structural Stability**
- **Regime IV — Extreme-Entropy Collapse**

we classify this instance as follows:

- At all times, **H_norm ≈ 0.99–1.0** and **KQ ≈ 0.005**.  
- Coherence is orders of magnitude smaller than the structural ceiling C (~0.839).  

Therefore the entire run belongs to:

> **Regime IV — Extreme-Entropy Collapse**  
> A persistent, single-phase collapse of global coherence under near-maximal entropy.

There are no excursions into Regime I+/I/II/III; the system is always in IV.

---

## 5. Geometric Relationships and Correlations

### 5.1 Full-run correlation matrix

Pearson correlations over all 1000 steps:

- corr(H_norm, KQ)   ≈ −0.999996  
- corr(H_norm, C)     ≈ −0.0601  
- corr(H_norm, kappa) ≈ +0.9828  
- corr(KQ, C)         ≈ +0.0627  
- corr(KQ, kappa)     ≈ −0.9828  
- corr(C, kappa)      ≈ −0.0508  

**Facts:**

- H_norm and KQ are **almost perfectly anti-correlated** (≈ −0.999996), as required by KQ = C·(1−H_norm) with slowly varying C.  
- H_norm and kappa are **strongly positively correlated** (≈ 0.983).  
- KQ and kappa are **strongly negatively correlated** (≈ −0.983).  
- C is only very weakly correlated with the other metrics.

**Interpretation:**

- Because the system is confined to a narrow band in C, KQ essentially serves as a **rescaled measure of (1 − H_norm)**.  
- The strong positive correlation between H_norm and kappa means that **as entropy increases slightly, curvature also increases slightly**. In this regime, higher turbulence is associated with tighter geometric stiffness.

The scatter plots (`H_norm` vs `KQ`, `kappa` vs `H_norm`) confirm these linear relationships as thin bands.

---

## 6. Practical Interpretation (QT viewpoint)

From the Quant-Trika perspective:

1. **Structure (C):**  
   - High and stable (~0.839).  
   - The underlying constraint graph / formula structure is **rich and intact**.

2. **Entropy (H_norm):**  
   - Almost maximal and extremely stable.  
   - The system essentially **never finds or maintains low-entropy basins**; it remains in persistent conflict/turbulence.

3. **Curvature (kappa):**  
   - Very high and slightly positively linked to entropy.  
   - The landscape is **stiff but not guiding toward coherent basins**; instead it locks the solver into a highly constrained, turbulent region.

4. **Coherence (KQ):**  
   - Almost completely suppressed (≈0.005).  
   - There are no significant coherence spikes; the solver never enters a globally ordered regime.

5. **Difficulty intuition for `am_9_9.shuffled-as.sat03-365`:**  

   - This instance represents a **pure extreme-entropy stress test** for the QT Complexity Engine.  
   - It probes the solver’s ability to **recover coherence from a nearly maximal-entropy, stiff geometric environment**.  
   - In practice, the current run shows **no recovery at all**: the engine remains in Regime IV throughout.

In short:

> `am_9_9.shuffled-as.sat03-365` is a **one-phase, extreme-entropy instance** with high structure and curvature but almost zero coherence. It is ideal for testing whether any future modifications of the engine can break out of a fully turbulent plateau.

---

## 7. Cross-Instance Comparison (with `alu4mul` and `C880mul`)

Using the global aggregates for the three analyzed instances:

- `alu4mul.miter.shuffled-as.sat03-344`:  
  - E_C ≈ 0.8782, E_H ≈ 0.5558, Var_H ≈ 0.2222, E_kappa ≈ 0.9867, K_QT ≈ 0.3893.  
  - **Regime profile:** multi-phase (I+ ↔ IV ↔ II/III) with extreme entropy swings.

- `C880mul.miter.shuffled-as.sat03-348`:  
  - E_C ≈ 0.8751, E_H ≈ 0.5373, Var_H ≈ 0.0078, E_kappa ≈ 0.9722, K_QT ≈ 0.4048.  
  - **Regime profile:** mid-entropy single manifold (II → III), no extreme phases.

- `am_9_9.shuffled-as.sat03-365`:  
  - E_C ≈ 0.8392, E_H ≈ 0.9940, Var_H ≈ 0.0000019, E_kappa ≈ 0.9959, K_QT ≈ 0.0050.  
  - **Regime profile:** pure Regime IV, extreme-entropy collapse with no phase changes.

**Global comparison table:**

| Instance ID                          | E_C    | E_H    | E_kappa | Var_H     | K_QT   | E_Beta | Max_Beta | Qualitative Regime Profile                                                   |
|--------------------------------------|--------|--------|---------|-----------|--------|--------|----------|-------------------------------------------------------------------------------|
| alu4mul.miter.shuffled-as.sat03-344 | 0.8782 | 0.5558 | 0.9867  | 0.2222    | 0.3893 | 5000   | 5000     | I+ ↔ IV ↔ II/III (strong phase switching, extreme entropy swings)            |
| C880mul.miter.shuffled-as.sat03-348 | 0.8751 | 0.5373 | 0.9722  | 0.0078    | 0.4048 | 5000   | 5000     | II → III (single mid-entropy manifold, no extreme phases)                    |
| am_9_9.shuffled-as.sat03-365        | 0.8392 | 0.9940 | 0.9959  | 0.0000019 | 0.0050 | 5000   | 5000     | IV only (persistent extreme-entropy collapse, no recovery of coherence)      |

**Δ-geometry interpretation:**

- `alu4mul` explores **both ends of the entropy spectrum**, alternating between ultra-coherent and ultra-chaotic regimes.  
- `C880mul` occupies a **moderate, mid-entropy corridor**, balancing structure and noise without extremes.  
- `am_9_9` is locked into an **extreme, high-entropy state** with negligible coherence.

Together, these three benchmarks form a **useful triad** for QT Complexity Engine evaluation:

1. **Phase-switching stress test** (`alu4mul`).  
2. **Mid-entropy navigation test** (`C880mul`).  
3. **Extreme-entropy collapse test** (`am_9_9`).

Any improved engine should ideally show **distinctive, measurable gains** in KQ behavior on each of these complementary regimes.

