# QT Complexity Engine Report — `alu4mul.miter.shuffled-as.sat03-344`

## 0. Instance & Inputs

**Instance ID:** `alu4mul.miter.shuffled-as.sat03-344`

**Archive contents:**
- `summary.json` — global aggregate metrics.
- `metrics.csv` — time-series for `C`, `H_norm`, `KQ`, `kappa` over 1000 steps.
- `timeseries.png` — time-series plot (not used as a primary data source, because full numeric series is available in CSV).
- `histograms.png` — histograms of metrics.
- `scatter.png` — scatter plots between metrics.

All quantitative analysis below is based **directly on `summary.json` and `metrics.csv` only**, without inferring unseen numbers from the images.

---

## 1. Sanity & Consistency Checks

From `summary.json`:
- `E_C` = 0.8782368599
- `E_H` = 0.5558012959
- `E_kappa` = 0.9866513497
- `Var_H` = 0.2222212233
- `K_QT` = 0.3892923693
- `E_Beta` = 5000.0
- `Max_Beta` = 5000.0

From `metrics.csv` (1000 rows):
- Columns: `C`, `H_norm`, `KQ`, `kappa`.
- `describe()` confirms that the CSV means match the JSON aggregates to numerical precision:
  - mean(C) ≈ 0.878237
  - mean(H_norm) ≈ 0.555801
  - mean(KQ) ≈ 0.389292
  - mean(kappa) ≈ 0.986651

### Canonical relation check

We test the Quant-Trika canonical relation

> **KQ ≈ C · (1 − H_norm)**

pointwise over all 1000 steps. If we define `KQ_pred = C * (1 − H_norm)`, the residual `KQ − KQ_pred` has:
- mean ≈ −3.9×10⁻¹⁷
- std ≈ 5.9×10⁻¹⁷
- min ≈ −2.0×10⁻¹⁶
- max ≈ 9.5×10⁻¹⁷

These are at floating-point roundoff scale, so **KQ is numerically identical to C·(1−H_norm)** in this instance. No inconsistencies detected.

### Range sanity

From `metrics.csv`:
- `C`   ∈ [0.875267, 0.881733]
- `H_norm` ∈ [0.000024, 0.990062]
- `KQ` ∈ [0.008762, 0.876434]
- `kappa` ∈ [0.985641, 0.988757]

All metrics stay in expected ranges: C and kappa are high and tightly concentrated; H_norm spans nearly the full [0,1] interval; KQ spans almost [0, max(C)].

No corrupted or obviously invalid values are present.

---

## 2. Global Metrics (JSON view)

Here is the global picture from `summary.json`:

- **Structural Quality (E_C) ≈ 0.878**  
  High and stable structural backbone. The instance has a strong, persistent pattern structure.

- **Entropy (E_H) ≈ 0.556 with Var_H ≈ 0.222**  
  The mean entropy is in the mid-range, but the variance is large. This already suggests that the run likely **spends time in both very low-entropy and very high-entropy states**, rather than hovering around the middle.

- **Curvature (E_kappa) ≈ 0.9867**  
  Very high curvature, with a very small spread (see Section 3). The landscape is stiff: moves are highly constrained by the underlying geometry.

- **Coherence (K_QT) ≈ 0.389**  
  Coherence is neither near zero nor close to the maximum (~0.88). It sits in an intermediate regime, which will turn out to be the average of very coherent phases and almost fully collapsed phases.

- **Beta control parameter: E_Beta = Max_Beta = 5000**  
  The control parameter β is pinned at its maximum for the entire run. There is no schedule or annealing in this instance.

**Interpretation:** globally, the instance is characterized by **high structure and curvature**, **strongly fluctuating entropy**, and an **intermediate average coherence** that arises from mixing very different dynamical regimes.

---

## 3. Time-Series Structure (CSV view)

### 3.1 Overall descriptive statistics

Over all 1000 steps:
- **C (structural quality)**
  - mean ≈ 0.878237
  - std ≈ 0.002077
  - min/max ≈ [0.87527, 0.88173]
  
  C is always high and varies only by about ±0.002: the structural backbone of the instance is stable.

- **H_norm (normalized entropy)**
  - mean ≈ 0.5558
  - std ≈ 0.4716
  - min ≈ 0.000024
  - max ≈ 0.990062
  
  Entropy explores almost the entire [0,1] range and has a very large standard deviation. This is a clear signal of **bimodality / phase switching** rather than small fluctuations.

- **KQ (coherence)**
  - mean ≈ 0.3893
  - std ≈ 0.4133
  - min ≈ 0.00876
  - max ≈ 0.87643
  
  Coherence also spans almost the full possible range and has a large standard deviation, mirroring the behavior of H_norm via the canonical relation.

- **kappa (curvature)**
  - mean ≈ 0.98665
  - std ≈ 0.000705
  - min/max ≈ [0.98564, 0.98876]
  
  Curvature stays high with very small variation: the geometric stiffness of the landscape is fairly constant.

### 3.2 Phase decomposition in time

Using the time index (0–999) from `metrics.csv`, we can identify three clear phases based on H_norm:

1. **Phase 1 — Ultra-low entropy, high coherence**  
   - Steps: **0–392** (393 steps)
   - Statistics:
     - mean(C) ≈ 0.87633
     - mean(H_norm) ≈ 0.00236
     - mean(KQ) ≈ 0.87426
     - mean(kappa) ≈ 0.98691
   - H_norm stays extremely close to 0; KQ stays near its structural upper bound (≈ C).  
   
   **Fact:** this is a **Regime I+ style** window: the solver lives in an ultra-coherent, almost noise-free state, with very stiff curvature.

2. **Phase 2 — Near-maximal entropy, coherence collapse**  
   - Steps: **393–924** (532 steps)
   - Statistics:
     - mean(C) ≈ 0.87990
     - mean(H_norm) ≈ 0.98686
     - mean(KQ) ≈ 0.01156
     - mean(kappa) ≈ 0.98618
   - H_norm is very close to 1.0 across the whole phase; KQ is suppressed to the ~10⁻² level.
   
   **Fact:** this is a **Regime IV** window: entropy is almost maximal and global coherence essentially collapses. Structure C is still high, but it is buried under turbulence.

3. **Phase 3 — Mid-entropy, intermediate coherence**  
   - Steps: **925–999** (75 steps)
   - Statistics:
     - mean(C) ≈ 0.87646
     - mean(H_norm) ≈ 0.39823
     - mean(KQ) ≈ 0.52743
     - mean(kappa) ≈ 0.98864
   - H_norm returns to a mid-range (~0.4) with some spread; KQ recovers to ≈0.53.
   
   **Fact:** this final phase is a **Regime II / III boundary**: moderate entropy with reasonably strong coherence.

### 3.3 Temporal narrative (facts and interpretation)

- **Fact:**
  - Early steps (0–392) show **H_norm ≈ 0** and **KQ ≈ 0.87**.
  - Mid-run steps (≈400–900) show **H_norm ≈ 0.98–0.99** and **KQ ≈ 0.01**.
  - Late steps (≈950–999) show **H_norm decreasing from ≈0.35 to ≈0.58** and **KQ between ≈0.37 and ≈0.68**.

- **Interpretation:**
  - The engine **starts in a very coherent basin**, with almost no entropy and near-maximal KQ.
  - It then undergoes a **sharp transition into a highly turbulent, high-entropy phase** where coherence collapses almost completely.
  - Finally, it **climbs out into a compromise regime** where entropy is lowered but not eliminated, and coherence stabilizes at an intermediate value.

This three-phase structure is what drives the large Var_H and Var(KQ).

---

## 4. Regime Classification (per phase and globally)

Using the qualitative regime labels:
- **Regime I+ — Ultra-Low Entropy Coherence**
- **Regime I — Low-Entropy Coherence**
- **Regime II — Mid-Entropy Oscillation**
- **Regime III — High-Entropy Structural Stability**
- **Regime IV — Extreme-Entropy Collapse**

we can classify:

- **Phase 1 (0–392):**  
  - H_norm ≈ 0.002, KQ ≈ 0.874.  
  - This is clearly **Regime I+ (Ultra-Low Entropy Coherence)**.

- **Phase 2 (393–924):**  
  - H_norm ≈ 0.987, KQ ≈ 0.011.  
  - This is **Regime IV (Extreme-Entropy Collapse)**.

- **Phase 3 (925–999):**  
  - H_norm ≈ 0.40, KQ ≈ 0.53.  
  - This falls between **Regime II and III**: mid-entropy with substantial coherence.

**Global regime (over the whole run):**  
Because more than half of the steps are in Regime IV and a substantial early chunk is in Regime I+, the global averages land in a **mid-regime mixture**. From the QT perspective, this instance should be tagged as:

> **Global regime:** **Mixed I+/IV/II** — a 3-phase run with long periods of extreme coherence, long periods of extreme entropy, and a final stabilization in a mid-entropy coherent basin.

---

## 5. Geometric Relationships and Correlations

### 5.1 Correlation structure (full run)

Pearson correlations over all 1000 steps:
- corr(H_norm, KQ) ≈ **−1.0** (−0.999999)
- corr(H_norm, C) ≈ **+0.84**
- corr(KQ, C) ≈ **−0.84**
- corr(KQ, kappa) ≈ **+0.56**
- corr(H_norm, kappa) ≈ **−0.56**

**Facts:**
- H_norm and KQ are almost perfectly anti-correlated, as expected from the canonical relation.
- C and kappa are both high and weakly to moderately correlated with the entropy/coherence pair.

**Interpretation:**
- Because C varies only slightly, KQ is effectively a **rescaled measure of (1 − H_norm)**.
- kappa is consistently high, with a modest positive correlation to KQ and negative to H_norm, meaning that slightly higher curvature tends to coincide with lower entropy and higher coherence.

### 5.2 Segment-wise correlations

- **Phase 1 (0–392, I+):**
  - corr(H_norm, KQ) ≈ −0.985  
    Even within the ultra-low-entropy window, tiny entropy fluctuations still produce predictable, opposite fluctuations in KQ.

- **Phase 2 (393–924, IV):**
  - corr(H_norm, KQ) ≈ −1.0 (−0.999997)  
    In the high-entropy plateau, KQ is almost a pure linear function of H_norm.

- **Phase 3 (925–999, II/III):**
  - corr(H_norm, KQ) ≈ −1.0 (−0.999999)  
    Again, KQ tracks (1 − H_norm) extremely tightly.

Across all phases, the **KQ–H_norm relation behaves exactly as the QT formula prescribes**, with no anomalies.

---

## 6. Practical Interpretation (QT viewpoint)

From the Quant-Trika perspective (structure–entropy–curvature–coherence):

1. **Structure (C):**
   - Always high (~0.876–0.882).  
   - The instance has a **strong, rigid structural backbone** that does not collapse even under high entropy.

2. **Entropy (H_norm):**
   - Explores almost the full range [0,1].  
   - The run is **not** trapped in a single turbulence level; it clearly moves between ordered and disordered regimes.

3. **Curvature (kappa):**
   - Very high (~0.986–0.989) and fairly stable.  
   - The search landscape is **stiff**: local moves are heavily constrained by geometry.

4. **Coherence (KQ):**
   - Mirrors the behavior of entropy, from ~0.876 in Phase 1 down to ~0.01 in Phase 2, then back up to ~0.53 in Phase 3.
   
   - **Interpretation:** coherence is not monotonically improving or degrading; instead, the engine **explores and transitions between basins** with radically different coherence levels.

5. **Difficulty intuition for `alu4mul.miter.shuffled-as.sat03-344`:**
   - The instance supports **very coherent solutions** (Phase 1) but also admits a **large high-entropy manifold** (Phase 2) where coherence collapses.
   - The presence of a stiff curvature and strong structure suggests a **complex but highly constrained SAT landscape**: the engine can lock into a very ordered regime but can also be driven into a turbulent plateau where entropy dominates.

From a practitioner’s point of view, this instance looks like:

> A structurally rigid but **dynamically multi-phase** landscape, where the solver must avoid or escape a wide high-entropy plateau to maintain high coherence.

---

## 7. Global Comparison Table (initial row)

This table is meant to be extended as more instances are analyzed. For now it contains a single row for `alu4mul.miter.shuffled-as.sat03-344`.

| Instance ID                          | E_C    | E_H    | E_kappa | Var_H   | K_QT   | E_Beta | Max_Beta | Dominant Regimes (phases)                 |
|--------------------------------------|--------|--------|---------|---------|--------|--------|----------|--------------------------------------------|
| alu4mul.miter.shuffled-as.sat03-344 | 0.8782 | 0.5558 | 0.9867  | 0.2222  | 0.3893 | 5000   | 5000     | I+ (0–392), IV (393–924), II/III (925–999) |

Future instances (e.g. other SAT benchmarks or MaxCut graphs) should be appended to this table to enable direct Δ-geometry comparison across problems.

