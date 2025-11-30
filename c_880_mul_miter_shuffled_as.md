# QT Complexity Engine Report — `C880mul.miter.shuffled-as.sat03-348`

## 0. Instance & Inputs

**Instance ID:** `C880mul.miter.shuffled-as.sat03-348`

**Archive contents (for this analysis):**
- `summary.json` — global aggregate metrics.
- `metrics.csv` — time-series for `C`, `H_norm`, `KQ`, `kappa` over 1000 steps.
- `timeseries.png` — time-series plots for all four metrics (used only qualitatively).
- `histograms.png` — histograms of `H_norm` and `kappa` (qualitative confirmation of CSV statistics).
- `scatter.png` — scatter plots (`C` vs `H_norm`, `kappa` vs `C`).

All quantitative statements below are based **directly on `summary.json` and `metrics.csv`**. The images are used only to confirm shapes and phase structure, not to invent numbers.

---

## 1. Sanity & Consistency Checks

From `summary.json`:

- `E_C`      = 0.875112863719
- `E_H`      = 0.537315754324
- `E_kappa`  = 0.972205478370
- `Var_H`    = 0.007792553500
- `K_QT`     = 0.404798709449
- `E_Beta`   = 5000.0
- `Max_Beta` = 5000.0

From `metrics.csv` (`count = 1000` rows):

- mean(C)      ≈ 0.875113
- mean(H_norm) ≈ 0.537316
- mean(KQ)     ≈ 0.404799
- mean(kappa)  ≈ 0.972205

These means match the JSON aggregates to numerical precision, so **the JSON summary is consistent with the CSV data**.

### 1.1 Canonical relation check

We test the Quant-Trika canonical relation

> **KQ ≈ C · (1 − H_norm)**

pointwise for all 1000 time steps. Defining `KQ_pred = C * (1 − H_norm)`, the residual `KQ − KQ_pred` has:

- mean ≈ −2.30×10⁻¹⁷
- std  ≈ 2.74×10⁻¹⁷
- min  ≈ −8.33×10⁻¹⁷
- max  ≈ 0.00×10⁰

These are at floating-point roundoff level (~10⁻¹⁷), therefore **KQ is numerically identical to C·(1−H_norm)** for this instance. No inconsistencies with the canonical relation.

### 1.2 Range sanity

From `metrics.csv`:

- `C`      ∈ [0.865998, 0.881575]
- `H_norm` ∈ [0.382795, 0.734286]
- `KQ`     ∈ [0.233453, 0.540886]
- `kappa`  ∈ [0.970998, 0.973173]

All metrics lie inside their expected physical ranges. There are no NaNs, infinities, or obviously corrupted entries.

---

## 2. Global Metrics (JSON view)

Using the `summary.json` values:

- **Structural Quality (E_C) ≈ 0.875**

  High structural quality: the configuration space explored by the engine is **well-organized and pattern-rich**. The system never drops into a structurally chaotic regime.

- **Entropy (E_H) ≈ 0.537, Var_H ≈ 0.0078**

  Entropy is **firmly mid-range**. Unlike the first instance (`alu4mul`), which explored almost the full entropy interval with huge variance, this run has **moderate fluctuations around a stable mid-entropy level**. The variance is small, signalling the absence of extreme low- or high-entropy phases.

- **Curvature (E_kappa) ≈ 0.972**

  Curvature is high (~0.972) with a very small spread (see Section 3). The search landscape is **geometrically stiff**: local moves are strongly constrained, but not as extremely as in `alu4mul`.

- **Coherence (K_QT) ≈ 0.405**

  Coherence is **moderately high (~0.40)**: the system maintains a non-trivial amount of meaningful global order despite mid-level entropy.

- **Beta control parameter: E_Beta = Max_Beta = 5000**

  As in the previous instance, β is locked at its maximum for the entire run. There is no annealing schedule here either.

**Interpretation:** Globally, `C880mul.miter.shuffled-as.sat03-348` lives in a **mid-entropy, mid-to-high coherence regime** with **consistently high structure and curvature**. There are no catastrophic collapses of order; instead, the engine explores a relatively narrow band of the complexity landscape.

---

## 3. Time-Series Structure (CSV view)

### 3.1 Overall descriptive statistics

From `metrics.csv` over the full 1000-step run:

- **C (structural quality)**  
  - mean ≈ 0.875113  
  - std  ≈ 0.004215  
  - min/max ≈ [0.865998, 0.881575]  
  
  C is always high (~0.866–0.882) and varies only by about ±0.004. The structural backbone is **stable and never breaks down**.

- **H_norm (normalized entropy)**  
  - mean ≈ 0.537316  
  - std  ≈ 0.088320  
  - min/max ≈ [0.382795, 0.734286]  
  - quantiles:  
    - 10% ≈ 0.430667  
    - 25% ≈ 0.453027  
    - 50% ≈ 0.533709  
    - 75% ≈ 0.590855  
    - 90% ≈ 0.673911  
  
  Entropy ranges from ≈0.383 to ≈0.734, with most mass between ≈0.45 and ≈0.59. This matches the histogram: **a mid-entropy band with moderate spread**, not a bimodal or extreme distribution.

- **KQ (coherence)**  
  - mean ≈ 0.404799  
  - std  ≈ 0.076878  
  - min/max ≈ [0.233453, 0.540886]  
  
  Coherence runs between ≈0.233 and ≈0.541. It is neither near-zero nor near C; instead it oscillates within a **mid-range corridor**, reflecting the mid-level entropy.

- **kappa (curvature)**  
  - mean ≈ 0.972205  
  - std  ≈ 0.000542  
  - min/max ≈ [0.970998, 0.973173]  
  
  Curvature stays high (~0.971–0.973) with small variability. The histogram confirms a **tight, unimodal distribution**.

### 3.2 Coarse phase decomposition

The time-series plots reveal gentle drifts and a noticeable late-time change, but **no sharp phase transitions comparable to `alu4mul`**. For a coarse analysis, we partition the run into three equal-ish segments:

1. **Segment 1 — Steps 0–349 (350 steps)**  
   - mean(C)   ≈ 0.870256  
   - mean(H)   ≈ 0.530288, std(H) ≈ 0.065091  
   - min/max H ≈ [0.439923, 0.664744]  
   - mean(KQ)  ≈ 0.408714, std(KQ) ≈ 0.056319  
   - mean(kappa) ≈ 0.971864  
  
   **Fact:** Entropy sits around ≈0.53 with moderate variability; coherence around ≈0.41. This is a **stable mid-entropy, mid-coherence regime**.

2. **Segment 2 — Steps 350–749 (400 steps)**  
   - mean(C)   ≈ 0.878035  
   - mean(H)   ≈ 0.504172, std(H) ≈ 0.065937  
   - min/max H ≈ [0.412525, 0.681249]  
   - mean(KQ)  ≈ 0.435250, std(KQ) ≈ 0.057077  
   - mean(kappa) ≈ 0.972183  
  
   **Fact:** Entropy actually **drops slightly** (to ≈0.50), while C increases to ≈0.878. Coherence improves to ≈0.435. This is the **most coherent part of the run**, but still mid-entropy.

3. **Segment 3 — Steps 750–999 (250 steps)**  
   - mean(C)   ≈ 0.877237  
   - mean(H)   ≈ 0.600185, std(H) ≈ 0.112195  
   - min/max H ≈ [0.382795, 0.734286]  
   - mean(KQ)  ≈ 0.350596, std(KQ) ≈ 0.097904  
   - mean(kappa) ≈ 0.972720  
  
   **Fact:** In the last quarter of the run entropy **rises to ≈0.60** with a wider spread; coherence correspondingly drops to ≈0.35. The timeseries plot shows a visible late increase in H_norm and a decline in KQ.

### 3.3 Temporal narrative

- **Fact:**  
  - There is **no ultra-low-entropy phase** (H_norm never approaches 0).  
  - There is **no extreme high-entropy phase** (H_norm never approaches 1).  
  - Entropy oscillates within [≈0.38, ≈0.73], with gentle drifts.  
  - Coherence tracks `(1 − H_norm)` as expected, remaining in [≈0.23, ≈0.54].  
  - Curvature slowly increases over time (see `kappa(t)`), but only within a narrow band.

- **Interpretation:**  
  - The engine explores a **single, broadly connected mid-entropy manifold** rather than jumping between radically different phases.  
  - Segment 2 (steps 350–749) represents a **locally optimal coherence window** within that manifold, where C is highest and H_norm is slightly reduced.  
  - Segment 3 indicates a **move into a noisier subregion** of the same landscape: entropy grows and coherence declines, but the underlying structure and curvature remain strong.

---

## 4. Regime Classification

Using the qualitative regimes:

- **Regime I+ — Ultra-Low Entropy Coherence**
- **Regime I  — Low-Entropy Coherence**
- **Regime II — Mid-Entropy Oscillation**
- **Regime III — High-Entropy Structural Stability**
- **Regime IV — Extreme-Entropy Collapse**

we can classify:

- **Segment 1 (0–349):**  
  - H_norm ≈ 0.53, KQ ≈ 0.41.  
  - This is firmly **Regime II (Mid-Entropy Oscillation)**.

- **Segment 2 (350–749):**  
  - H_norm ≈ 0.50, KQ ≈ 0.44, with slightly higher C.  
  - Also **Regime II**, but at its more coherent, better-structured edge.

- **Segment 3 (750–999):**  
  - H_norm ≈ 0.60, KQ ≈ 0.35.  
  - This leans toward **Regime III (High-Entropy Structural Stability)** — entropy is higher but structure and curvature remain strong.

**Global regime (whole run):**

> **Regime II → III trajectory**  
> The run starts and stays mostly in Regime II, then drifts toward Regime III in the final quarter, without ever visiting Regime I+/I or Regime IV.

The contrast with `alu4mul` is sharp: that instance alternated between I+ and IV, whereas `C880mul` **never leaves the mid-band**.

---

## 5. Geometric Relationships and Correlations

### 5.1 Full-run correlations

Pearson correlations over all 1000 steps:

- corr(H_norm, KQ) ≈ -0.999681
- corr(H_norm, C)   ≈ 0.274873
- corr(KQ, C)       ≈ -0.250742
- corr(H_norm, kappa) ≈ 0.634414
- corr(KQ, kappa)     ≈ -0.622686
- corr(C, kappa)      ≈ 0.621354

**Facts:**

- H_norm and KQ are **almost perfectly anti-correlated** (≈ −0.9997), as expected from the canonical formula.
- Both C and kappa are **positively correlated with H_norm** and **negatively with KQ**.
- C and kappa themselves are **moderately strongly correlated** (≈ 0.62).

**Interpretation:**

- Because KQ is exactly C·(1−H_norm), any increase in H_norm at roughly fixed C inevitably decreases KQ; the near-perfect anti-correlation is a structural identity, not just an empirical trend.
- The positive correlation between H_norm and kappa suggests that **regions with slightly higher entropy also exhibit slightly higher curvature**. In other words, **the geometry becomes a bit stiffer as the landscape gets noisier**, but this effect is mild.
- The scatter plots (`C vs H_norm`, `kappa vs C`) confirm a **clustered, banded structure** rather than isolated clouds, consistent with mid-entropy oscillatory dynamics.

### 5.2 Segment-wise correlations (H_norm vs KQ)

Even when looking inside each segment separately:

- Segment 1 (0–349): corr(H_norm, KQ) ≈ -0.999867  
- Segment 2 (350–749): corr(H_norm, KQ) ≈ -0.999783  
- Segment 3 (750–999): corr(H_norm, KQ) ≈ -0.999996  

The anti-correlation remains **almost perfect in every segment**. This confirms that **no hidden redefinition** of KQ is used in any part of the run; it is consistently derived from C and H_norm.

---

## 6. Practical Interpretation (QT viewpoint)

From the Quant-Trika perspective:

1. **Structure (C):**  
   - Always high; increases slightly from Segment 1 to 2 and stays high in Segment 3.  
   - The instance has a **rigid, well-formed structural skeleton** that does not undergo qualitative changes.

2. **Entropy (H_norm):**  
   - Stays in a mid-range band throughout the run.  
   - Segment 2 achieves the **best compromise** (slightly lower entropy) while keeping structure high.

3. **Curvature (kappa):**  
   - High and slowly increasing over time.  
   - The landscape is **consistently stiff**, with only gentle geometric evolution.

4. **Coherence (KQ):**  
   - Moderately high overall (≈0.40).  
   - Highest in Segment 2 (~0.44), lowest in the late high-entropy Segment 3 (~0.35).  
   - There are no coherence spikes or collapses — just **smooth oscillations**.

5. **Difficulty intuition for `C880mul.miter.shuffled-as.sat03-348`:**  

   - This instance represents a **balanced, mid-entropy SAT landscape**:  
     - It is **less extreme** than `alu4mul` (no ultra-ordered or ultra-chaotic regimes).  
     - The solver moves within a **single coherent but noisy manifold**, rather than jumping between incompatible basins.
   - From a solver design viewpoint, this looks like a case where **fine-tuning local search inside a stiff, well-structured geometry** is more important than handling dramatic phase transitions.

In short:

> `C880mul` is a **mid-entropy, mid-to-high coherence benchmark** with strong geometry and no catastrophic turbulence. It is structurally rich but dynamically tame compared to `alu4mul`.

---

## 7. Cross-Instance Comparison (with `alu4mul.miter.shuffled-as.sat03-344`)

Using the recomputed metrics for the first instance (`alu4mul`) from its own archive:

- `alu4mul.miter.shuffled-as.sat03-344`:
  - E_C ≈ 0.8782
  - E_H ≈ 0.5558
  - E_kappa ≈ 0.9867
  - Var_H ≈ 0.2222
  - K_QT ≈ 0.3893

The global comparison:

| Instance ID                          | E_C    | E_H    | E_kappa | Var_H   | K_QT   | E_Beta | Max_Beta | Qualitative Regime Profile                                   |
|--------------------------------------|--------|--------|---------|---------|--------|--------|----------|----------------------------------------------------------------|
| alu4mul.miter.shuffled-as.sat03-344 | 0.8782 | 0.5558 | 0.9867  | 0.2222  | 0.3893 | 5000   | 5000     | I+ ↔ IV ↔ II/III (strong phase switching, extreme entropy swings) |
| C880mul.miter.shuffled-as.sat03-348 | 0.8751 | 0.5373 | 0.9722  | 0.0078  | 0.4048 | 5000   | 5000     | II → III (single mid-entropy manifold, no extreme phases)       |

**Interpretation of Δ-geometry:**

- **Entropy dynamics:**  
  - `alu4mul` has **very large Var_H** and visits both near-zero and near-one entropy — a strongly multi-phase system.  
  - `C880mul` has **small Var_H** and stays in the mid-range — a single-phase, oscillatory system.

- **Coherence landscape:**  
  - `alu4mul` alternates between **near-maximal coherence** and **almost total coherence collapse**.  
  - `C880mul` keeps coherence in a **moderate corridor**, never close to 0 or to its structural maximum.

- **Curvature & structure:**  
  - Both have high C and high kappa, but `alu4mul` is slightly stiffer on average.  
  - Geometrically, both instances are rigid; their main difference is **how entropy interacts with that rigidity**.

From a Quant-Trika complexity viewpoint:

> **`alu4mul`** is a **multi-phase, extreme-entropy benchmark** — a good test of whether a solver can survive and exploit violent coherence/entropy transitions.  
> **`C880mul`** is a **mid-entropy, geometrically stiff benchmark** — a good test of whether a solver can efficiently navigate a single, noisy but well-structured manifold without relying on phase changes.

Both should be kept in the benchmark suite because they probe **different aspects of the complexity geometry**.

