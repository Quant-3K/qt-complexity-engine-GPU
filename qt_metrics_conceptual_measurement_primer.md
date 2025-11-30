# Quant‑Trika Metric System — Conceptual Primer

*Author: Artem Brezgin & the Quant‑Trika Framework (C) 2025, Spanda Foundation*

---

## 0. Purpose of This Document

This document is meant to be a **conceptual introduction** to the core Quant‑Trika (QT) metrics used throughout the SAT benchmark reports:

- **C(x)** — Structural Quality
- **H_norm(x)** — Normalized Entropy
- **κ(x)** — Computational Curvature
- **KQ(x)** — Coherence Metric
- Auxiliary statistics: **Var_H**, correlations, and **Regime labels**

The goal is not only to say *what the numbers are*, but to clarify:

1. **Where these metrics come from conceptually** — what they are trying to capture.
2. **How they are constructed in principle** — which ingredients go into their definitions.
3. **Why we use exactly this set** — why this quartet (C, H_norm, κ, KQ) is sufficient to describe the landscape.
4. **How to read any future QT report** — independently of the specific SAT instance.

This is a “lens manual”: once you understand what each metric means, the tables and plots become transparent.

---

## 1. The Quant‑Trika Measurement Philosophy

Quant‑Trika treats any law‑governed system (including SAT solvers) as a **dynamical process in a configuration space**. At each step, the system occupies a state \(x\) and can move to neighboring states according to some transition rule (e.g., local flips, heuristic moves, probabilistic jumps).

Instead of focusing on traditional complexity proxies (runtime, conflict count, branching factor), Quant‑Trika asks a deeper question:

> **What is the *shape* of the landscape in which the solver is moving?**

To answer this, we attach to each state \(x\) a quadruple of observables:

\[
\big(C(x),\; H_{\text{norm}}(x),\; \kappa(x),\; K_Q(x)\big)
\]

Each of these captures a **different aspect of how structured, noisy, and dynamically stable** the local neighborhood of \(x\) is.

- **C(x)**: How much *structural order* or *meaningful pattern* is present in the current assignment.
- **H_norm(x)**: How much *uncertainty*, *conflict*, or *instability* is present in the local transition possibilities.
- **κ(x)**: How the "geometry" of the landscape bends around \(x\) — whether local moves reinforce or disrupt order.
- **KQ(x)**: The resulting *coherence* — the effective quality of the state once both structure and entropy are taken into account.

These four together define what we call the **QT landscape**: a 4D field over configuration space.

---

## 2. Structural Quality C(x)

### 2.1 Intuitive Meaning

**Structural Quality** \(C(x)\) is intended to measure **how ordered, consistent, and “sharp”** the current state is, *regardless of entropy*.

In a SAT context, think of:

- how many clauses are satisfied,
- how robustly they are satisfied (not sitting on a knife‑edge of being flipped),
- how aligned the variable assignments are with the underlying logical structure of the formula.

High \(C(x)\) means:

- the assignment exhibits a **clear structural pattern**,
- many constraints are simultaneously well‑satisfied,
- small perturbations tend to preserve large parts of the structure.

Low \(C(x)\) means:

- the state is **structurally noisy** or fragmented,
- satisfied clauses are scattered and fragile,
- there is no strong global pattern holding the solution together.

### 2.2 Conceptual Construction

The exact implementation of \(C(x)\) can vary between experiment families, but conceptually it is built from:

1. **Clause satisfaction profile** — how many clauses are satisfied vs. unsatisfied.
2. **Local stability of satisfaction** — how many clauses would flip status under small perturbations.
3. **Pattern / contrast measures** — whether satisfied clauses cluster into consistent blocks rather than being random.

In general terms, \(C(x)\) is normalized to lie in **[0, 1]**, where:

- \(C(x) \approx 1\): strongly ordered, structurally robust state.
- \(C(x) \approx 0\): almost no detectable structure, or structure that is immediately destroyed by local moves.

### 2.3 Why C(x) Is Necessary

We track \(C(x)\) because **entropy alone cannot tell us whether a state is meaningful**.

You can have:

- Low entropy but also low structure (a trivial, degenerate configuration).
- High structure but also high entropy (rigid but internally conflicted systems).

\(C(x)\) isolates the **pure structural content** of the state, independent of how noisy or conflicted its neighborhood is.

---

## 3. Normalized Entropy H_norm(x)

### 3.1 Intuitive Meaning

**Normalized entropy** \(H_{\text{norm}}(x)\) measures **how uncertain the local future is**, given the current state.

Instead of asking “how many clauses are satisfied,” we ask:

> *If the solver follows its transition rules from \(x\), how many qualitatively different next configurations are plausible, and how evenly are they distributed?*

High \(H_{\text{norm}}(x)\) means:

- the local neighborhood is **turbulent**,
- many competing moves exist with similar heuristic scores,
- the engine has no clear gradient to follow.

Low \(H_{\text{norm}}(x)\) means:

- the local neighborhood is **ordered**,
- a small subset of moves clearly dominate others,
- the engine effectively “knows” where it should go.

### 3.2 Conceptual Construction

Conceptually, \(H_{\text{norm}}(x)\) is built from a **probability distribution over possible next moves or local configurations**.

1. From state \(x\), consider the **set of admissible local transitions** (e.g., flipping variables allowed by the solver’s policy).
2. Assign each move a **score** (e.g., expected clause improvement, heuristic energy, or internal solver weight).
3. Turn these scores into a **probability distribution** \(p_i(x)\) over moves (e.g., via softmax or normalization).
4. Compute the **Shannon entropy** \(H(x) = -\sum_i p_i(x) \log p_i(x)\).
5. Normalize by the maximum possible entropy for this neighborhood: \(H_{\text{norm}}(x) = H(x) / H_{\max}\), so that \(H_{\text{norm}}(x) \in [0, 1].\)

Interpretation:

- \(H_{\text{norm}}(x) \approx 0\): one move is overwhelmingly preferred → “sharp” gradient, low uncertainty.
- \(H_{\text{norm}}(x) \approx 1\): many moves with similar probabilities → “flat” or confused landscape.

### 3.3 Why H_norm(x) Is Necessary

Where \(C(x)\) measures **what is**, \(H_{\text{norm}}(x)\) measures **what could happen next**.

You can have a highly structured state (high C) but:

- if every local move keeps you in conflict, \(H_{\text{norm}}\) will be high → **frozen chaos**,
- if there is a clear path to improve, \(H_{\text{norm}}\) will be low → **stable order with a clear gradient**.

Thus, \(H_{\text{norm}}\) captures the “turbulence” of the solver’s local decision landscape.

---

## 4. Computational Curvature κ(x)

### 4.1 Intuitive Meaning

**Curvature** \(\kappa(x)\) aims to capture **how the landscape bends around the state**.

If we think of \(KQ(x)\) as a kind of "height" or "potential" over configuration space, curvature answers:

> *When we make small steps around \(x\), do we discover a bowl, a ridge, or a saddle?*

High curvature (in absolute terms) indicates that **small changes in configuration produce strong changes in coherence**.

In the QT SAT experiments, we focus on a **normalized, bounded curvature** \(\kappa(x) \in [0, 1]\), interpreted as:

- \(\kappa(x) \approx 1\): the system is strongly regulated — the geometry resists random drift, local moves have sharply structured effects.
- \(\kappa(x)\) lower: the geometry is looser, more fragile, or fragmenting.

### 4.2 Conceptual Construction

Conceptually, \(\kappa(x)\) is derived from **second‑order variation of coherence** in the local neighborhood of \(x\):

1. Take a local set of neighbors \(\{x_i\}\) reachable from \(x\) by small moves.
2. Evaluate \(KQ(x_i)\) for each neighbor.
3. Estimate a local **curvature-like quantity** from how these \(KQ\) values change with direction and distance.
4. Normalize into \([0, 1]\) to make different instances comparable.

The exact numerical formula may differ by implementation, but the **semantic invariants** are:

- high \(C(x)\) typically pushes \(\kappa(x)\) up (rigid structure stiffens geometry),
- high \(H_{\text{norm}}(x)\) tends to push \(\kappa(x)\) down (entropy destabilizes geometry),
- \(\kappa(x)\) therefore encodes the **tension between structure and entropy** in geometric form.

### 4.3 Why κ(x) Is Necessary

Even if we know \(C\) and \(H_{\text{norm}}\), we still don’t know:

- whether local improvements in coherence accumulate into stable basins, or
- whether they remain fragile and easily destroyed.

Curvature \(\kappa(x)\) tells us **how robust the landscape is** under repeated local moves.

Two states can have the same (C, H_norm) but differ:

- high \(\kappa\): local improvements reinforce each other, forming attractors;
- lower \(\kappa\): local improvements cancel or fragment, forming turbulent structures.

Thus \(\kappa\) transforms the static (C, H_norm) picture into a **geometric dynamical** picture.

---

## 5. Coherence Metric KQ(x)

### 5.1 Definition and Canonical Form

In the QT SAT experiments, **coherence** is defined by the canonical formula:

\[
KQ(x) = C(x) \cdot \big(1 - H_{\text{norm}}(x)\big).
\]

This is not an arbitrary choice; it is designed to encode several essential constraints:

1. **If entropy is maximal** (\(H_{\text{norm}} = 1\)), then \(KQ = 0\) regardless of structure → complete incoherence.
2. **If entropy is minimal** (\(H_{\text{norm}} = 0\)), then \(KQ = C\) → coherence reduces to pure structural quality.
3. **If structure is absent** (\(C = 0\)), then \(KQ = 0\) regardless of entropy → random chaos without structure.

Thus \(KQ\) is a **multiplicative coupling** of structure and entropy: coherence survives only when there is both *enough structure* and *sufficient entropy suppression*.

By construction, \(KQ(x) \in [0, 1]\).

### 5.2 Intuitive Meaning

\(KQ(x)\) answers a very specific question:

> *Given the structural quality of this state and the turbulence in its neighborhood, how much effective order actually survives?*

High \(KQ(x)\) means:

- the state is structurally rich (high C),
- entropy in its local neighborhood is low (low H_norm),
- the system can preserve and extend its internal patterns.

Low \(KQ(x)\) means:

- either structure is weak, or
- entropy is overwhelming, or
- both.

In practice, low \(KQ\) regimes correspond to **globally incoherent** landscapes, where local improvements constantly get undone by turbulence.

### 5.3 Role of KQ in the Landscape

While \(C\), \(H_{\text{norm}}\), and \(\kappa\) are “raw ingredients,” \(KQ\) is the **central integrative observable**.

- Time series of \(KQ(t)\) tell us about **stability, attractors, and phases**.
- Distributions of \(KQ\) across samples distinguish between **coherent, oscillatory, and collapsed** regimes.
- Cross‑instance averages \(E[KQ]\) provide a **comparative measure of global problem hardness** in QT terms.

In other words, \(KQ\) is the primary measure of **how well a system converts structural potential into stable order under entropy pressure**.

---

## 6. Auxiliary Metrics: Var_H, Correlations, and Regimes

### 6.1 Var_H — Entropy Variance

The **variance of entropy** over time,

\[
\text{Var}_H = \text{Var}[H_{\text{norm}}(t)],
\]

captures **how much the system moves between different entropy zones**:

- Low Var_H + low E_H → stable order with little turbulence.
- Low Var_H + high E_H → *persistent chaos* (always turbulent).
- High Var_H → **intermittent turbulence**, with bursts of order and bursts of chaos.

In practice, Var_H helps us distinguish between:

- systems that are *consistently* low‑entropy (Regime I, I+),
- systems that *alternate* (Regime II),
- systems that are *consistently* high‑entropy (Regime III–IV).

### 6.2 Correlation Structure

We also track correlations such as:

- corr(C, H_norm)
- corr(C, κ)
- corr(H_norm, κ)
- corr(KQ, H_norm)

These reveal **causal tensions** in the landscape:

- Strong negative corr(H_norm, κ) → entropy actively suppresses curvature.
- corr(C, κ) ≈ 0 → curvature is not simply a function of structure.
- corr(KQ, H_norm) ≈ −1 → by definition, as entropy rises, coherence collapses.

The correlation matrix therefore encodes the **internal logic of the landscape**: which forces dominate and how they interact.

### 6.3 Regime Labels

Finally, we use the combination of \(E_C\), \(E_H\), \(E_\kappa\), \(\text{Var}_H\), and \(E[KQ]\) to assign **regime labels**, such as:

- **Regime I+ — Ultra‑Low Entropy Coherence**
- **Regime I — Low‑Entropy Coherence**
- **Regime II — Mid‑Entropy Oscillation**
- **Regime III — High‑Entropy Structural Stability**
- **Regime IV — Extreme‑Entropy Collapse**

These regimes are not merely descriptive; they are defined by inequalities in the observables (e.g., thresholds in E_H, E_KQ, Var_H, and specific correlation patterns).

Regimes provide a **coarse‑grained, human‑readable summary** of the landscape type without losing the detailed metric information.

---

## 7. Why This Particular Quartet of Metrics?

One might ask: why not track dozens of observables? Why focus on exactly these four (C, H_norm, κ, KQ) plus a small set of auxiliaries?

The answer is **parsimony** plus **coverage**.

The metric system is designed to satisfy:

1. **Completeness (for our use‑case)**:
   - C(x) → structural order.
   - H_norm(x) → local uncertainty / turbulence.
   - κ(x) → geometric regulation / fragility.
   - KQ(x) → effective coherence under entropy.

   Together they describe **structure, noise, geometry, and emergent order**.

2. **Interpretability**:
   - Each metric has a clear conceptual meaning.
   - Their combination matches intuitive notions: “this instance is structured but chaotic”, “this one is clean and coherent”, etc.

3. **Comparability across instances**:
   - All metrics are normalized to [0, 1].
   - Averages and variances are dimensionless and comparable for different SAT families.

4. **Theoretical grounding**:
   - KQ = C(1 − H_norm) is consistent with the basic QT principle that **coherence exists only where structure and entropy suppression co‑exist**.
   - Curvature is treated as the **geometric manifestation** of the structure–entropy tension.

In this sense, the quartet (C, H_norm, κ, KQ) forms a **minimal, but theoretically motivated, basis** for describing QT landscapes.

---

## 8. How to Read Any QT Report Using This Primer

When you open a QT report for a new SAT instance, you can now interpret the metrics as follows:

1. **Look at E_C**:
   - High (~0.85–0.90): strong structural backbone.
   - Low: poor structural organization.

2. **Look at E_H and Var_H**:
   - Low E_H, low Var_H: stable, ordered landscape.
   - Mid E_H, high Var_H: oscillatory, mixed regime.
   - High E_H, low Var_H: persistent chaos.

3. **Look at E_κ**:
   - High: geometry is stiff and regulated.
   - Lower: geometry is fragile or fragmented.

4. **Look at K_QT (mean KQ)**:
   - High (~0.7–0.8): globally coherent.
   - Mid (~0.2–0.6): partial coherence with significant turbulence.
   - Very low (< 0.1): coherence collapse.

5. **Check correlations**:
   - Strong negative corr(H_norm, κ) → entropy “wins” over curvature.
   - Weak corr(C, κ) → curvature not trivially tied to structure.

6. **Read the Regime label** as a compact summary of all the above.

This way, every new instance — regardless of name or origin — can be placed into a **shared geometric language of complexity**.

---

## 9. Conceptual Summary

To summarize, the QT metric system provides a way to **rephrase computational hardness as geometry**:

- **C(x)** tells you *how much structure there is*.
- **H_norm(x)** tells you *how turbulent the local future is*.
- **κ(x)** tells you *how the landscape bends and resists random drift*.
- **KQ(x)** tells you *how much of the structure survives under entropy*.

Auxiliary statistics like Var_H, correlations, and regime labels then translate these raw observables into a **phase portrait** of the instance.

In this view, complexity is no longer just “how long the algorithm runs,” but:

> **how structure, entropy, and geometry conspire to either support or destroy coherence in the space of possibilities.**

This is the core vision behind Quant‑Trika as a measurement framework for computational landscapes.

