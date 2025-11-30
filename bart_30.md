# QT Complexity Engine Report — `bart30.shuffled`

## 0. Instance & Inputs

**Instance ID:** `bart30.shuffled`  
**Source archive:** `5000GPU20251128_184659_bart30.shuffled.zip`

**Files used in this analysis:**
- `summary.json` — global aggregate metrics.
- `metrics.csv` — time-series for `C`, `H_norm`, `KQ`, `kappa` over 1000 steps.
- `timeseries.png` — qualitative check of temporal shapes.
- `histograms.png` — qualitative check of distributions of `H_norm` and `kappa`.
- `scatter.png` — qualitative check of pairwise relations (e.g., `C`–`H_norm`, `kappa`–`C`).

All **numbers** below come **directly from `summary.json` and `metrics.csv`**.  
Images are used only to confirm the shapes / regimes, never as a numeric source.

---

## 1. Sanity & Consistency Checks

From `summary.json`:

- `E_C`      = 0.785957426727
- `E_H`      = 0.607548205703
- `E_kappa`  = 0.822481622458
- `Var_H`    = 0.004297222206
- `K_QT`     = 0.308438214355
- `E_Beta`   = 5000.0
- `Max_Beta` = 5000.0

From `metrics.csv` (`count = 1000` rows):

- mean(C)      ≈ 0.785957
- mean(H_norm) ≈ 0.607548
- mean(KQ)     ≈ 0.308438
- mean(kappa)  ≈ 0.822482

These means match the JSON aggregates to numerical precision, so **the JSON summary is consistent with the raw CSV data**.

### 1.1 Canonical relation check

We test the canonical Quant‑Trika relation

> **KQ ≈ C · (1 − H_norm)**

pointwise for all 1000 steps by defining  
`KQ_pred = C * (1 − H_norm)` and looking at the residual `KQ − KQ_pred`.

Residual statistics:

- mean residual ≈ -2.58e-17
- std residual  ≈ 2.81e-17
- min residual  ≈ -8.33e-17
- max residual  ≈ 0.00e+00

These values are at pure floating‑point roundoff scale (~10⁻¹⁷), therefore **KQ is numerically identical to C·(1−H_norm)** for this instance. There is no redefinition or drift of KQ.

### 1.2 Range sanity

From `metrics.csv`:

- `C`      ∈ [0.748410, 0.821399]
- `H_norm` ∈ [0.443248, 0.759305]
- `KQ`     ∈ [0.190320, 0.441398]
- `kappa`  ∈ [0.810031, 0.837117]

All metrics lie within their expected physical ranges, with no NaNs or infinities.

---

## 2. Global Metrics (JSON view)

Using `summary.json`:

- **Structural Quality (E_C) ≈ 0.786**

  Structure is **high**: even though E_C is slightly below the algebraic instances (`alu4mul`, `C880mul`), it still indicates a **strongly organized, non‑random constraint structure**.

- **Entropy (E_H) ≈ 0.608, Var_H ≈ 0.0043**

  Entropy is **mid–high** (~0.61) with a **moderate variance**. The system explores both more ordered windows (H_norm ≈ 0.44–0.5) and more turbulent states (H_norm up to ≈0.76), but does not touch the extreme 0 or 1 limits.

- **Curvature (E_kappa) ≈ 0.822**

  Curvature is **high (~0.82)**, indicating that the search landscape is **stiff and geometrically constrained**, but less stiff than the `alu4mul` / `am_9_9` cases.

- **Coherence (K_QT) ≈ 0.308**

  Coherence is **moderate (~0.31)**. This is clearly above the near‑zero collapse of `am_9_9`, but significantly below the more coherent C880mul / alu4mul runs.

- **Beta control parameter: E_Beta = Max_Beta = 5000**

  As with the other instances, β is pinned to 5000 for the whole run. There is no annealing or schedule.

**High‑level picture:** `bart30` has a **rigid structural skeleton** with **mid–high entropy** and **only moderate global coherence**. It is structurally ordered but dynamically noisy.

---

## 3. Time‑Series Structure (CSV view)

### 3.1 Overall descriptive statistics

From `metrics.csv`:

- **C (structural quality)**  
  - mean ≈ 0.785957  
  - std  ≈ 0.014017  
  - min/max ≈ [0.748410, 0.821399]  

  → C stays high (≈0.75–0.82) with std ≈ 0.014. The structural backbone is stable; it never collapses into a random regime.

- **H_norm (normalized entropy)**  
  - mean ≈ 0.607548  
  - std  ≈ 0.065586  
  - min/max ≈ [0.443248, 0.759305]  
  - quantiles:  
    - 10% ≈ 0.537994  
    - 25% ≈ 0.574113  
    - 50% ≈ 0.611214  
    - 75% ≈ 0.645916  
    - 90% ≈ 0.680982  

  → Entropy lives primarily between ≈0.57 and ≈0.65, with tails down to ≈0.44 and up to ≈0.76. This matches a **mid–high entropy band with occasional excursions**.

- **KQ (coherence)**  
  - mean ≈ 0.308438  
  - std  ≈ 0.051920  
  - min/max ≈ [0.190320, 0.441398]  

  → Coherence ranges from ≈0.19 to ≈0.44. It never approaches 0 (full collapse) or C (near‑perfect order) — it stays in a **middle corridor**.

- **kappa (curvature)**  
  - mean ≈ 0.822482  
  - std  ≈ 0.005028  
  - min/max ≈ [0.810031, 0.837117]  

  → Curvature is high and tight (≈0.81–0.84). The geometry is stiff with only gentle modulation.

### 3.2 Extremal coherence events

- **Maximum KQ (best coherence)**  
  - index = 141  
  - C ≈ 0.793752  
  - H_norm ≈ 0.443909  
  - KQ ≈ 0.441398  
  - kappa ≈ 0.823352  

  Here H_norm ≈ 0.444 is near the lower end of its range, so KQ rises to ≈0.441. Structure and curvature remain high.

- **Minimum KQ (worst coherence)**  
  - index = 237  
  - C ≈ 0.790711  
  - H_norm ≈ 0.759305  
  - KQ ≈ 0.190320  
  - kappa ≈ 0.822281  

  Here H_norm ≈ 0.759 is near the upper tail, and KQ correspondingly falls to ≈0.190. Again, C and κ change very little — **entropy alone is what crushes coherence**.

### 3.3 Coarse phase decomposition

To detect possible phases, we split the run into four equal segments of 250 steps each and compute statistics:

1. **Segment 1 — steps 0–249**

   - mean(C)   ≈ 0.790141  
   - mean(H)   ≈ 0.559294, std(H) ≈ 0.089800  
   - min/max H ≈ [0.443248, 0.759305]  
   - mean(KQ)  ≈ 0.348347, std(KQ) ≈ 0.071626  
   - mean(kappa) ≈ 0.822436  

   → This is the **most coherent segment**: entropy is relatively low (~0.56 on average) and KQ is highest (~0.35). Entropy variance is also largest here, containing both low‑entropy dips and the global max.

2. **Segment 2 — steps 250–499**

   - mean(C)   ≈ 0.793860  
   - mean(H)   ≈ 0.630417, std(H) ≈ 0.035193  
   - min/max H ≈ [0.572370, 0.724715]  
   - mean(KQ)  ≈ 0.293252, std(KQ) ≈ 0.026972  
   - mean(kappa) ≈ 0.825543  

   → Entropy rises to ≈0.63 and becomes more concentrated. Coherence drops to ≈0.293. Curvature κ peaks here (~0.826), indicating a **very stiff geometric region under higher entropy**.

3. **Segment 3 — steps 500–749**

   - mean(C)   ≈ 0.782227  
   - mean(H)   ≈ 0.631621, std(H) ≈ 0.037034  
   - min/max H ≈ [0.556266, 0.725855]  
   - mean(KQ)  ≈ 0.287977, std(KQ) ≈ 0.027780  
   - mean(kappa) ≈ 0.821975  

   → Similar entropy (~0.632) and coherence (~0.288) to Segment 2, but with slightly weaker structure and curvature. This looks like **continued exploration of a similar high‑entropy manifold**.

4. **Segment 4 — steps 750–999**

   - mean(C)   ≈ 0.777601  
   - mean(H)   ≈ 0.608861, std(H) ≈ 0.056011  
   - min/max H ≈ [0.506154, 0.745553]  
   - mean(KQ)  ≈ 0.304178, std(KQ) ≈ 0.043776  
   - mean(kappa) ≈ 0.819973  

   → Entropy relaxes slightly (~0.609), structure and curvature soften a bit, and coherence partially recovers (~0.304), but not to Segment‑1 levels.

**Temporal narrative:**

- Start in a **relatively coherent, still stiff basin** (Segment 1).
- Move into a **more turbulent, still rigid high‑entropy corridor** (Segments 2–3) where coherence is consistently lower.
- End in a **slightly less turbulent, slightly softer region** (Segment 4), with modest KQ recovery.

### 3.4 Regime occupancy fractions

Using simple thresholds:

- Fraction of low‑entropy steps (H_norm < 0.5): **0.092** ≈ 9.2%  
- Fraction of high‑entropy steps (H_norm > 0.7): **0.072** ≈ 7.2%  
- Fraction of high‑coherence steps (KQ > 0.36): **0.132** ≈ 13.2%  
- Fraction of low‑coherence steps (KQ < 0.25): **0.114** ≈ 11.4%  

Most of the time the system resides in **mid‑entropy, mid‑coherence states**, with non‑negligible but minority excursions into more coherent or more collapsed regimes.

---

## 4. Regime Classification

We use the same coarse regimes:

- **Regime I** — Low‑entropy coherence (low H, high KQ).  
- **Regime II** — Mid‑entropy oscillation.  
- **Regime III** — High‑entropy structural stability.  
- **Regime IV** — Extreme‑entropy collapse.

For `bart30`:

- Global E_H ≈ 0.608 (mid–high).  
- Global K_QT ≈ 0.308 (moderate).  
- Structural quality and curvature stay high and tight.

Segment‑wise:

- Segment 1 leans toward **Regime II** (lower entropy, higher KQ).  
- Segments 2–3 lean toward **Regime III** (higher entropy, still structurally stiff).  
- Segment 4 is between II and III.

**Overall classification:**

> `bart30.shuffled` is a **Regime II–III hybrid**: a mid–high entropy, structurally rigid instance with moderate coherence and no extreme phase transitions.

---

## 5. Geometric Relationships and Correlations

Full‑run Pearson correlations:

- corr(C, kappa)      ≈ 0.975  
- corr(H_norm, KQ)    ≈ -0.995  
- corr(C, KQ)         ≈ 0.088  
- corr(H_norm, kappa) ≈ 0.080  
- corr(C, H_norm)     ≈ 0.013  

**Facts:**

- C and κ are **strongly positively correlated** (~0.98) → curvature is almost a direct geometric echo of structural quality.  
- H_norm and KQ are **almost perfectly anti‑correlated** (~−0.995), as expected from KQ = C·(1−H_norm) when C changes slowly.  
- C and KQ have only a **weak positive correlation** (~0.09).  
- H_norm and κ are only weakly correlated (~0.08).

**Interpretation:**

- Geometry (`C`, `kappa`) forms a **rigid backbone** that evolves slowly and coherently.  
- Entropy injects turbulence on top of this backbone and is the **dominant driver of coherence loss**.  
- Structural improvements alone do not guarantee high coherence; they must be accompanied by reduced entropy.

The scatter plots in `scatter.png` are consistent with this: a tight linear band for `H_norm` vs `KQ`, and a narrow, positive cluster for `C` vs `kappa`.

---

## 6. Practical Interpretation (QT viewpoint)

From the Quant‑Trika perspective:

1. **Structure (C):**  
   - High and relatively stable, with mild erosion over time.  
   - The SAT instance has a **strong, non‑random constraint graph**.

2. **Entropy (H_norm):**  
   - Sits mainly in a **mid–high band (~0.56–0.65)** with occasional excursions.  
   - This means the solver constantly faces a substantial amount of conflict and uncertainty.

3. **Curvature (kappa):**  
   - High (≈0.82) and tightly coupled to C.  
   - The landscape is **narrow and stiff**: moves are strongly constrained, but that stiffness doesn’t automatically yield coherence.

4. **Coherence (KQ):**  
   - Moderate (~0.31) on average, sensitive to entropy spikes.  
   - Roughly 13% of the time the system enjoys comparatively high coherence (>0.36), and 11% of the time it is in significantly degraded coherence (<0.25).

5. **Instance difficulty intuition:**  

   - `bart30` is **harder** (less coherent) than `alu4mul` and `C880mul`, because it sustains a higher entropy level and lower KQ on a similarly rigid structure.  
   - It is **much easier** than `am_9_9`, which lives in a near‑maximal entropy collapse regime with KQ ≈ 0.005.  
   - Conceptually, `bart30` is a **“dirty but structured” landscape**: the geometry is clean and stiff, but entropy injects enough conflict to keep global coherence only modest.

---

## 7. Cross‑Instance Comparison (4‑way)

Using the JSON summaries for all four analyzed instances:

| Instance ID                            | E_C   | E_H    | E_kappa | Var_H        | K_QT  | Qualitative regime profile                                                          |
|----------------------------------------|-------|--------|---------|--------------|-------|-------------------------------------------------------------------------------------|
| alu4mul.miter.shuffled-as.sat03-344    | 0.8782 | 0.5558 | 0.9867 | 0.2222 | 0.3893 | Strong structure; very large Var_H; multi‑phase I+ ↔ IV ↔ II/III (extreme swings). |
| C880mul.miter.shuffled-as.sat03-348    | 0.8751 | 0.5373 | 0.9722 | 0.0078 | 0.4048 | High structure, mid entropy, high coherence; stable II → III manifold.             |
| bart30.shuffled                        | 0.7860 | 0.6075 | 0.8225 | 0.0043 | 0.3084 | Mid–high entropy, rigid geometry, moderate coherence; II–III hybrid.              |
| am_9_9.shuffled-as.sat03-365           | 0.8392 | 0.9940 | 0.9959 | 0.000002 | 0.0050 | Pure IV: extreme‑entropy collapse, near‑zero coherence, no phase recovery.         |

**Δ‑geometry summary:**

- `alu4mul` explores **both extremes** of entropy and coherence (huge Var_H).  
- `C880mul` is a **well‑behaved, mid‑entropy coherent instance**.  
- `bart30` is **noisier and less coherent** than C880mul/alu4mul, but far from full collapse.  
- `am_9_9` is the **degenerate extreme‑entropy limit**, almost zero coherence.

Together, they span a wide spectrum of QT regimes, and `bart30` fills the important **intermediate “structured but noisy” slot** in this spectrum.

