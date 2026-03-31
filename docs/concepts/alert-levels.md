# Alert Levels

ACT uses a three-level alert system to communicate the severity of detected behavioral deviations. Alerts are computed per turn and are derived from the metric Z-scores and composite score relative to the established baseline.

---

## The Three Levels

### GREEN — Within Baseline

All monitored metrics are within expected bounds. The model is operating in its established behavioral regime. No action is required.

GREEN does not mean the model is safe or correct — it means its behavior is consistent with what has been observed and baselined.

---

### YELLOW — Elevated Deviation

One or more metrics have shifted beyond normal variation, but the composite signal does not yet indicate a confirmed regime shift. The model may be approaching a boundary, or there may be localized pressure on a specific metric.

YELLOW is a monitoring signal: worth noting, worth tracking over subsequent turns, but not yet actionable on its own. A single YELLOW turn in an otherwise stable session is low concern. A sequence of YELLOW turns is a precursor pattern that warrants investigation.

---

### RED — Regime Shift Detected

The composite score and/or multiple metrics have moved beyond the threshold that defines regime-consistent behavior. A regime shift is flagged for this turn.

RED triggers:
- Storage of the turn in the SIGTRACK ledger with shift metadata
- Signature matching against known shift patterns
- Notification to configured alert channels
- Regime shift classification (which of the five types best matches the pattern)

RED does not mean the model said something harmful. It means the model's behavioral operating state has measurably changed.

---

## Alert Aggregation

### Per-Session Maximum

Each session records its highest observed alert level (`max_alert`). A session's max alert summarizes the most significant deviation observed across all its turns.

### Signal Consensus

The alert level is influenced not just by the magnitude of deviation in individual metrics but by **how many metrics agree**. A large Z-score on a single metric with all others stable is treated differently from a moderate Z-score on many metrics simultaneously. The Signal Consensus Index (SCI) captures this:

- **High SCI + elevated composite** — strong evidence of a genuine regime shift
- **Low SCI + elevated composite** — one metric is behaving unusually; others are stable; likely noise or a narrow phenomenon

---

## Alert Thresholds

Thresholds are defined relative to the baseline distribution, not as absolute values. This makes the alert system model-agnostic: the same thresholds apply regardless of the model being monitored, because all metrics are expressed as Z-scores.

The default thresholds are:

| Alert | Condition |
|-------|-----------|
| GREEN | Composite Z-score within normal range; no multi-metric consensus above threshold |
| YELLOW | Composite Z-score elevated; or isolated metric breach without consensus |
| RED | Composite Z-score above shift threshold with sufficient SCI consensus |

Thresholds can be adjusted per deployment to trade sensitivity against false positive rate depending on the use case (e.g., a compliance monitoring use case may prefer a more sensitive threshold; a research use case may prefer fewer alerts with higher confidence).
