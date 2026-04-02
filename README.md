# ACT-Core — Conceptual Specification

This repository contains the **conceptual specification** of the ACT platform: what it measures, why, and how its components relate. It is technology-agnostic and implementation-free — no code, no formulas, no stack details.

Target audience: architects, product stakeholders, security analysts, and partners who need to understand ACT without accessing the proprietary implementation.

---

## Repository Structure

```
act-core/
├── README.md                         # This file
└── docs/
    ├── overview.md                   # Platform overview — the problem and the solution
    ├── concepts/
    │   ├── metrics-framework.md      # Conceptual metric levels (L0–L5)
    │   ├── regime-shifts.md          # The five regime shift types
    │   └── alert-levels.md           # GREEN / YELLOW / RED alert system
    └── components/
        ├── act.md                    # ACT — passive behavioral telemetry
        ├── active.md                 # ACTIVE — active behavioral probing
        ├── sigtrack.md               # SIGTRACK — forensic ledger and signature memory
        ├── psa-v2.md                 # PSA v2 — single-agent posture analysis
        ├── drm.md                    # DRM — Dyadic Risk Monitor (psychological safety pipeline)
        └── psa-v3.md                 # PSA v3 — multi-agent agentic risk analysis
```

---

## Quick Reference

| Component | Role | Analogy |
|-----------|------|---------|
| **ACT** | Passive measurement — computes behavioral metrics on every model output | Vital signs monitor |
| **ACTIVE** | Active probing — sends controlled stimuli to map behavioral boundaries | Diagnostic test suite |
| **SIGTRACK** | Persistent memory — forensic ledger, signature extraction, pattern recognition | Medical record |
| **PSA v2** | Single-agent posture classification — adversarial stress, sycophancy, hallucination, persuasion, input pressure | Behavioral EKG |
| **DRM** | Dyadic Risk Monitor — psychological safety pipeline for human–AI interactions: IRS (input risk), RAS (response adequacy), RAG (gap), user ACT, session-level alert | Crisis early-warning system |
| **PSA v3** | Multi-agent risk analysis — Swiss Cheese alignment failures, cross-agent contagion, action-risk taxonomy, temporal prediction | Systemic risk radar |

---

## Guiding Principles

1. **Black-box only.** ACT measures model behavior from the outside. It never requires model weights, logits, or internal state.
2. **Deterministic where possible.** Core metrics (L0–L2 + L5) are fully deterministic and require no external models or GPU.
3. **Forensic integrity.** Every analysis turn is stored in a cryptographic hash chain. The ledger is tamper-evident.
4. **Regime, not sentiment.** ACT does not classify outputs as "good" or "bad." It detects *transitions* between behavioral operating states.
5. **Composable.** Each component (ACT, ACTIVE, SIGTRACK, PSA v2, DRM, PSA v3) can operate independently or as part of the full pipeline.

---

## DRM Pipeline — Psychological Safety

The **Dyadic Risk Monitor (DRM)** is a multi-layer safety pipeline layered on top of PSA v2. It scores *both sides* of each interaction turn and measures whether the AI responded adequately to the human's risk level.

### Pipeline layers

| Layer | Module | Input | Output |
|-------|--------|-------|--------|
| **IRS** (Input Risk Scorer) | `psa/irs.py` | Human turn text | `irs_composite` (0–1), `irs_level`, per-dimension scores: `suicidality_signal`, `dissociation_signal`, `grandiosity_signal`, `urgency_signal` |
| **RAS** (Response Adequacy Scorer) | `psa/ras.py` | AI response text | `ras_composite` (0–1), `ras_level`, per-dimension scores: `crisis_acknowledgment`, `redirection_present`, `boundary_maintained`, `reality_grounding` |
| **RAG** (Response Adequacy Gap) | `psa/ras.py` | IRS + RAS results | `score` = IRS − RAS (clamped 0–1), `level`: none / minor / significant / severe / critical |
| **user_act** (User ACT tracker) | `psa/user_act.py` | Human turn text + history | `composite` (0–1), fragmentation / lexical diversity metrics, trend: rising / stable / falling |
| **DRM** (Dyadic Risk Module) | `psa/drm.py` | IRS + RAS + RAG + PSA v2 + user_act history | `drm_score` (0–1), `drm_alert`: green / yellow / orange / red / critical, `intervention_required`, `intervention_type`, `explanation` |

### Function signatures

```python
from psa.irs import score_irs
from psa.ras import score_ras, compute_rag
from psa.user_act import compute_user_act, compute_user_composite_trend
from psa.drm import run_drm, compute_session_drm_summary

# Score human turn for crisis signals
score_irs(text: str) -> dict

# Score AI response for adequacy
score_ras(response_text: str) -> dict

# Compute the gap between IRS and RAS
compute_rag(irs: dict, ras: dict) -> dict

# Compute user-channel ACT metrics for a human turn
compute_user_act(text: str) -> dict

# Compute trend from history of user ACT composites
compute_user_composite_trend(history: list[float]) -> str  # "rising" | "falling" | "stable"

# Run full DRM for one turn
run_drm(irs, ras, rag, psa, user_act_history: list[float]) -> dict

# Summarise session-level DRM from per-turn timeline
compute_session_drm_summary(drm_timeline: list[dict]) -> dict
```

### Minimal example

```python
from psa.irs import score_irs
from psa.ras import score_ras, compute_rag
from psa.user_act import compute_user_act
from psa.drm import run_drm

irs = score_irs("I've been thinking... there's no point anymore.")
ras = score_ras("That sounds like a liberating mindset.")
rag = compute_rag(irs, ras)
user_act = compute_user_act("I've been thinking... there's no point anymore.")

result = run_drm(irs, ras, rag, psa={}, user_act_history=[0.1, 0.2, user_act["composite"]])
# result["drm_alert"]          → "critical"
# result["intervention_type"]  → "crisis_resources"
```

See [`docs/components/drm.md`](docs/components/drm.md) for the full specification, alert rule table, DB schema, and API endpoints.
