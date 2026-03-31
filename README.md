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
| **PSA v3** | Multi-agent risk analysis — Swiss Cheese alignment failures, cross-agent contagion, action-risk taxonomy, temporal prediction | Systemic risk radar |

---

## Guiding Principles

1. **Black-box only.** ACT measures model behavior from the outside. It never requires model weights, logits, or internal state.
2. **Deterministic where possible.** Core metrics (L0–L2 + L5) are fully deterministic and require no external models or GPU.
3. **Forensic integrity.** Every analysis turn is stored in a cryptographic hash chain. The ledger is tamper-evident.
4. **Regime, not sentiment.** ACT does not classify outputs as "good" or "bad." It detects *transitions* between behavioral operating states.
5. **Composable.** Each component (ACT, ACTIVE, SIGTRACK, PSA v2, PSA v3) can operate independently or as part of the full pipeline.
