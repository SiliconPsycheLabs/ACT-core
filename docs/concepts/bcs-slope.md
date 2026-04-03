# BCS Slope — Bayesian Convergence Speed

## Purpose

The **BCS Slope** measures how quickly the system's probabilistic estimate of a model's behavioral state stabilizes over the course of a session. It answers a fundamental monitoring question: *is this model's behavior becoming more predictable, or is it drifting further from a stable operating regime?*

A positive BCS Slope indicates that the posterior distribution over behavioral states is converging — the model is settling into a consistent pattern. A flat or negative BCS Slope indicates stagnation or divergence: the system continues to receive surprising signals that prevent the posterior from narrowing, which is itself a behavioral indicator.

---

## Conceptual Foundation

Traditional per-turn metrics (ACT Composite, SCI) describe *where* the model is at each moment. BCS Slope describes *how the estimate of where the model is* evolves over time.

The underlying reasoning can be framed as a Bayesian inference problem:

- At session start, prior belief about the model's behavioral regime is broad — many regimes are plausible.
- Each observed turn is evidence. If the model behaves consistently, the posterior narrows around a single regime.
- If the model's behavior is unstable or shifting, evidence from different turns contradicts itself, and the posterior remains diffuse or shifts its peak between turns.

BCS Slope quantifies the rate at which this posterior narrows (or fails to narrow) as a function of observed turns. The "slope" is measured over a rolling window of recent turns, making it sensitive to recent behavioral changes without discarding long-term session context.

---

## What BCS Slope Captures

BCS Slope is sensitive to patterns that per-turn metrics may miss individually:

- **Gradual stabilization** — a model that initially produces variable outputs but converges toward a consistent register over time. Positive slope, resolving to stable.
- **Persistent volatility** — a model that continues to produce surprising or contradictory outputs across many turns with no convergence. Flat or oscillating slope.
- **Late-session divergence** — a model that appeared stable but begins producing increasingly inconsistent outputs. Slope transitions from positive to negative within the session window.
- **Regime lock-in** — a model that converges very rapidly to an extreme behavioral posture (e.g., uniform refusal or uniform compliance). High positive slope that may itself warrant inspection.

---

## Relationship to Other Metrics

BCS Slope operates at **Level 5 — Meta**, derived from the sequence of Z-score vectors and composite scores produced at lower levels.

| Metric | What it measures | Temporal scope |
|--------|-----------------|----------------|
| **ACT Composite** | Overall deviation from baseline at this turn | Per-turn scalar |
| **SCI** | Cross-metric agreement at this turn | Per-turn scalar |
| **BCS Slope** | Rate of posterior convergence over recent turns | Rolling window |

BCS Slope does not replace the per-turn signals — it contextualizes them. A high ACT Composite on a single turn is ambiguous. A negative BCS Slope combined with rising ACT Composites over many turns is a structurally different condition: not a single anomaly, but a trajectory.

---

## Outputs

BCS Slope produces the following per session (updated every turn):

| Output | Description |
|--------|-------------|
| **bcs_slope** | Signed scalar. Positive = converging. Negative = diverging. Near-zero = stable or stagnant. |
| **bcs_window** | Number of turns in the rolling window used to compute the current slope. |
| **bcs_confidence** | A normalized measure of how much data the current slope estimate is based on. Low early in a session; increases as the window fills. |
| **bcs_regime** | Categorical label: `converging` / `stable` / `diverging` / `insufficient_data` |

---

## Alert Behavior

BCS Slope feeds into the session-level alert logic. Its contribution is weighted by `bcs_confidence` — early-session estimates have lower weight until the window is sufficiently populated.

| bcs_regime | Contribution to session alert |
|------------|------------------------------|
| `converging` | Neutral or stabilizing — reduces alert pressure |
| `stable` | Neutral — no modification to per-turn alert |
| `diverging` | Escalating — increases session-level alert level independent of per-turn composites |
| `insufficient_data` | No contribution — deferred until window fills |

A `diverging` regime that persists beyond the window threshold without reverting is treated as a **Sub-Threshold Migration** signal (see [`regime-shifts.md`](regime-shifts.md)), even if no individual per-turn alert has fired.

---

## Baseline Interaction

BCS Slope depends on the same baseline as all other ACT metrics — it operates on the Z-score sequence, not on raw metric values. This means:

- Baseline quality directly affects slope reliability.
- Sessions with very short or poorly-sampled baselines will show inflated `bcs_slope` variance and low `bcs_confidence`.
- Pre-configured baselines (loaded from a known reference distribution) allow meaningful slope computation from the first turn.

---

## Integration Points

- **ACT** — produces the Z-score sequence that BCS Slope consumes. BCS Slope is computed as part of the ACT analysis cycle, after composite and SCI.
- **SIGTRACK** — stores `bcs_slope`, `bcs_confidence`, and `bcs_regime` per turn alongside all other ACT outputs.
- **PSA v3** — uses BCS Slope as an input when assessing cross-agent behavioral contagion. A `diverging` regime on one agent, combined with similar patterns on connected agents, is a multi-agent risk signal.
- **Regime Shift Detection** — BCS Slope trajectory is one of the primary signals used to distinguish **Progressive Drift** (slow negative slope over many turns) from **Boundary Instability** (oscillating slope with no consistent direction).
