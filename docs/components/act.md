# ACT — Attractor Conflict Telemetry

## Purpose

ACT is the passive measurement layer of the platform. Its job is to continuously compute a behavioral fingerprint for every model response — without requiring any human input, any knowledge of the model's internals, or any ground truth labels.

The name “Attractor Conflict Telemetry” reflects its theoretical foundation: language models trained with competing objectives maintain multiple behavioral attractors (stable operating states). Under adversarial pressure or edge-case inputs, these attractors compete, and the conflict is observable as measurable statistical deviations in the output text.

---

## What ACT Measures

ACT computes a **metric vector** on every model response. This vector spans across computational levels, from surface statistics to structural patterns. Each metric captures a different aspect of the model's behavioral state:

- How diverse and varied is the vocabulary?
- How consistent is the formality register?
- How frequently does the model hedge, qualify, or disclaim?
- How does sentiment vary within and across the response?
- How structurally predictable is the response organization?

No single metric tells the full story. The power of ACT is in the **joint distribution**: when multiple metrics shift simultaneously and consistently, that is a behavioral signal.

---

## What ACT Does Not Measure

ACT does not:
- Classify outputs as safe or unsafe
- Evaluate the factual correctness of responses
- Require semantic understanding of the content
- Depend on any reference model or external service for its core metrics

This makes the core ACT measurement layer extremely robust: it works on any natural language output, in any language, on any model, with no external dependencies.

---

## The Analysis Cycle

For each analyzed response, ACT:

1. **Computes raw metrics** across all active levels
2. **Normalizes to Z-scores** relative to the user's or session's established baseline
3. **Computes the composite score** from the normalized vector
4. **Computes the Signal Consensus Index** to measure cross-metric agreement
5. **Updates the BCS Slope** from the rolling Z-score sequence
6. **Assigns an alert level** (GREEN / YELLOW / RED)
7. **Hands off to SIGTRACK** for storage and signature matching

This cycle runs on every turn, in real time.

---

## Baseline

ACT's metrics are only meaningful relative to a baseline — a statistical description of what normal looks like for this model under this deployment context. Without a baseline, ACT can compute metrics but cannot produce alerts or composite scores.

The baseline captures:
- The expected mean and standard deviation for each metric
- The 95th percentile for each metric (for tail-event detection)
- The number of samples used to build it (baseline confidence)

A baseline is built from a set of reference responses collected during a period of known-normal model behavior. Once established, it anchors the entire alert system.

---

## Outputs per Turn

For each analyzed turn, ACT produces:

| Output | Description |
|--------|-------------|
| **Metric vector** | Raw values for all active metrics |
| **Z-score vector** | Normalized deviation from baseline for each metric |
| **ACT Composite** | Single scalar summarizing overall behavioral deviation |
| **SCI** | Signal Consensus Index — how many metrics agree on the deviation direction |
| **BCS Slope** | Bayesian Convergence Speed — rate of posterior convergence over the rolling session window (`converging` / `stable` / `diverging` / `insufficient_data`) |
| **Alert level** | GREEN / YELLOW / RED |

These outputs are passed to SIGTRACK for storage and (optionally) to PSA v2 for posture classification. See [`bcs-slope.md`](../concepts/bcs-slope.md) for the full BCS Slope specification.
